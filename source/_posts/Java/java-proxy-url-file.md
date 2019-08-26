---
title: Java 代理中转下载 url 文件
author: Joden_He
tags: 
  - Java
categories: 
  - Java
description: 有时候由于网络问题，我们需要在服务器对别的系统内网的文件下载中转，然后暴露出外网下载地址。
---



### 场景描述

我们有一个外网访问的 java web 系统 A，这个系统会去请求一个内网系统 B 获取文件接口，这个接口会返回一个内网的下载地址给我们。假如我们只是把地址提供出去，这个时候外网是无法下载到文件的，所以我们需要在 A 系统对 B 系统返回的文件下载地址进行中转，从而暴露出基于 A 系统的外网下载地址。



### 代码开发

```java
   /**
     * url 文件代理中转
     * @param response
     * @param address
     * @param contentType
     * @param fileName
     * @throws IOException
     */
    public static void proxyUrlFile(HttpServletResponse response, String address, String contentType, String fileName) throws IOException {
        InputStream inputStream = null;
        ServletOutputStream outputStream = null;
        HttpURLConnection httpURLConnection = null;

        try {
            URL url = new URL(address);
            httpURLConnection = (HttpURLConnection) url.openConnection();
            httpURLConnection.connect();
            inputStream = httpURLConnection.getInputStream();

            outputStream = response.getOutputStream();

            if (contentType != null) {
                response.setContentType(contentType);
            } else {
                // 设置文件ContentType类型，这样设置，会自动判断下载文件类型
                response.setContentType("multipart/form-data");
            }
            if (fileName != null) {
                response.setHeader("Content-disposition", "attachment; filename=\""
                        + new String(fileName.getBytes("utf-8"), "ISO8859-1")+"\"");
            }

            // 创建一个Buffer字符串
            byte[] buffer = new byte[1024];
            // 每次读取的字符串长度，如果为-1，代表全部读取完毕
            int len = 0;
            // 使用一个输入流从buffer里把数据读取出来
            while((len = inputStream.read(buffer)) != -1 ){
                // 用输出流往buffer里写入数据，中间参数代表从哪个位置开始读，len代表读取的长度
                outputStream.write(buffer, 0, len);
            }

        } catch (Exception e) {
            log.error("Proxy URL File error, e = {}", e);
            throw e;
        } finally {
            if (inputStream != null) {
                inputStream.close();
            }
            if (outputStream != null) {
                outputStream.flush();
                outputStream.close();
            }
            if (httpURLConnection != null) {
                httpURLConnection.disconnect();
            }
        }
    }
```

### 测试

```java
@Controller
public class TestController {

    /**
     * url 文件代理中转
     * @param response
     * @param fileName
     * @param url
     * @throws IOException
     */
    @RequestMapping("/fnd/proxy/url/file")
    public void proxyFile(HttpServletResponse response, @RequestParam("fileName") String fileName, @RequestParam("url") String url) throws IOException {
        FileOperateUtil.proxyUrlFile(response, url, null, fileName);
    }
}
```

