**例子**

```java
package com.xuanyuan.ffmpeg.server.task;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.*;
import java.util.ArrayList;
import java.util.List;

/**
 * @ClassName: MpegTask
 * @description: TODO
 * @author: LEE
 * @create: 2020-05-27 10:48
 * @Version 1.0
 **/
public class MpegTask implements Runnable {
    private static final Logger logger = LoggerFactory.getLogger(MpegTask.class);

    private String url;//源视频链接
    private String transcoPath;//转码存放目录

    private String dirPath;//文件目录名

    public MpegTask(String url,String transcoPath,String dirPath){
        this.url=url;
        this.transcoPath=transcoPath;
        this.dirPath=dirPath;
    }

    @Override
    public void run() {
        try {
            System.out.println("进入转码.......");
            // 创建命令集合
            List<String> commandList = new ArrayList<String>();

            //判断系统，只分windows与非windows,
            String system=System.getProperty("os.name").toLowerCase();
            if(system.indexOf("windows")>-1){
                commandList.add("cmd");
                commandList.add("/c");  // 执行结束后关闭
            }
            //默认设置每块分片120秒,分片默认以play.m3u8命名 以文件名生成文件夹区分每个整体视频
            commandList.add("ffmpeg");
            commandList.add("-i");
            commandList.add(url);
            commandList.add("-c:v");
            commandList.add("libx264");
            commandList.add("-hls_time");
            commandList.add("120");
            commandList.add("-hls_list_size");
            commandList.add("0");
            commandList.add("-c:a");
            commandList.add("aac");
            commandList.add("-strict");
            commandList.add("-2");
            commandList.add("-f");
            commandList.add("hls");
            commandList.add(transcoPath+ File.separator+dirPath+File.separator+"play.m3u8");

            // ProcessBuilder是一个用于创建操作系统进程的类，它的start()方法用于启动一个进行
            ProcessBuilder processBuilder = new ProcessBuilder(commandList);
            // 启动进程
            Process process = processBuilder.start();
            doWaitFor(process);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public int doWaitFor(Process process) {
        int exitValue = -1; // returned to caller when p is finished
        try(InputStreamReader inr=new InputStreamReader(process.getInputStream(),"gbk");
            InputStreamReader err = new InputStreamReader(process.getErrorStream(),"gbk");
            BufferedReader inbr = new BufferedReader(inr);
            BufferedReader errbr = new BufferedReader(err);) {
            boolean finished = false; // Set to true when p is finished
            while (!finished) {
                try {
                    StringBuffer sb = new StringBuffer();
                    String line = "";
                    while ((line=inbr.readLine())!=null) {
                        sb.append(line);
                    }
                    if (sb.length()>0) {
                        System.out.println("in"+sb.toString());
                        logger.info("in"+sb.toString());
                    }

                    sb = new StringBuffer();
                    while ((line=errbr.readLine())!=null) {
                        sb.append(line);
                    }
                    if (sb.length()>0) {
                        System.out.println("err"+sb.toString());
                        logger.error("err"+sb.toString());
                    }

                    exitValue = process.exitValue();
                    System.out.println(exitValue);
                    finished = true;
                } catch (IllegalThreadStateException e) {
                    Thread.currentThread().sleep(500);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return exitValue;
    }
}
```

