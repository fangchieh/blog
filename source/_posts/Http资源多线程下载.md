---
title: Http资源多线程下载
date: 2019-07-25 10:54:51
tags: 笔记
---
#### 实现http资源多线程下载的小案例：

```java
import java.io.IOException;
import java.io.InputStream;
import java.io.RandomAccessFile;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;

public class MultiThreadDownload {

    private String path;

    private int threadNum;

    public MultiThreadDownload(String path, int threadNum) {

        this.path = path;

        this.threadNum = threadNum;

    }

    public void startDownload() throws IOException {

        URL url = new URL(path);

        HttpURLConnection conn =(HttpURLConnection)url.openConnection();

        conn.setRequestMethod("GET");

        conn.setConnectTimeout(5000);

        conn.connect();

        if (conn.getResponseCode()==200){

            //1.获取要下载资源的大小。
            int contentLength = conn.getContentLength();

            //2.创建并设置同等大小的临时文件
            RandomAccessFile raf = new RandomAccessFile("D:\\"+getFileName(),"rwd");

            raf.setLength(contentLength);

            raf.close();

            // 计算每个线程下载的大小，先不考虑有余数的情况
            int block = contentLength / threadNum;

            for (int i = 0; i < threadNum;i++){

                //计算线程下载的开始位置和结束位置
                int startIndex = block * i;

                int endIndex = block * (i + 1) - 1;

                //解决有余数的情况。方法：直接写死最后一个线程的结束位置
                if (i == threadNum-1){

                    endIndex = contentLength-1;

                }

                new DownThread(i,startIndex,endIndex).start();

            }
        }

    }

    private String getFileName() {

        String fileName = path.substring(path.lastIndexOf("/") + 1);

        return fileName;
    }

    //使用成员内部类可以无条件访问外部类的所有成员属性和成员方法（包括private成员和静态成员）。
    class DownThread extends Thread{

        private int ThreadId;

        private int start;

        private int end;

        public DownThread(int threadId, int start, int end) {

            this.ThreadId = threadId;

            this.start = start;

            this.end = end;

        }

        @Override
        public void run() {

            super.run();

            try {
                //再次发送Http资源请求。
                URL url = new URL(path);

                HttpURLConnection conn = (HttpURLConnection)url.openConnection();

                conn.setRequestMethod("GET");

                conn.setConnectTimeout(5000);

                conn.setDoInput(true);
                // Server通过请求头中的Range: bytes=0-xxx来判断是否是做Range请求，
                // 如果这个值存在而且有效，则只发回请求的那部分文件内容，响应的状态码变成206，表示Partial Content，并设置Content-Range。
                // 如果无效，则返回416状态码，表明Request Range Not。
                conn.setRequestProperty("Range","bytes="+start+"-"+end);

                conn.connect();
                // 请求部分数据,响应码为206
               if (conn.getResponseCode()== 206){

                   InputStream in = conn.getInputStream();

                   RandomAccessFile raf = new RandomAccessFile("D:\\"+getFileName(),"rwd");

                   byte[] by = new byte[1024];

                   int len = 0;

                   // 把文件的写入位置移动至start=startIndex
                   raf.seek(start);

                   while((len=in.read(by))!=-1){

                       raf.write(by,0,len);

                   }

                   raf.close();

                   in.close();


               }else {

                   System.out.println("部分资源请求失败！");

               }


            } catch (MalformedURLException e) {

                e.printStackTrace();

            } catch (IOException e) {

                e.printStackTrace();

            }

        }
    }

    public static void main(String[] args) throws IOException {

        new MultiThreadDownload("http://pic25.nipic.com/20121112/9252150_150552938000_2.jpg",4).startDownload();

    }

}
```

