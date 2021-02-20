---
title: "https绕过证书认证请求 Get或Post请求（证书过期，忽略证书）"
date: 2020-04-13T14:46:09+08:00
description: "javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target"
categories:
 ["https"]
tags : [
"https"]
featured_image:
author: "ipoo"
KeyName : "https绕过证书认证请求,Get请求,httpPost请求"

---


<!-- MarkdownTOC -->

- [报错信息](#%E6%8A%A5%E9%94%99%E4%BF%A1%E6%81%AF)
- [解决：](#%E8%A7%A3%E5%86%B3%EF%BC%9A)
    - [postman方式](#postman%E6%96%B9%E5%BC%8F)
    - [java请求](#java%E8%AF%B7%E6%B1%82)

<!-- /MarkdownTOC -->

## 报错信息

```txt
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
```
**javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target**

1. 如果证书过期，配置容器为http请求，就可搞定。
2. 如果postman请求遇到此情况:**Could not get any response**


![postman请求提示](https://oss.ipooli.com/images/blog/could.png)

## 解决：
### postman方式
1. 打开postman --->  File  --->  Settings 
![配置](http://oss.ipooli.com/images/blog/byssl2.png)
2. 关闭图片红框里的SSL..选项为OFF

这样就可以了.

### java请求
直接贴代码了。

```java
/**
     * 采用绕过验证的方式处理https请求
     * @param url
     * @param json
     * @return
     */
    public static String doGet(String url, JSONObject json) {
        String body = "";
        SSLContext sslcontext = null;
        try {
            //设置协议http和https对应的处理socket链接工厂的对象
            sslcontext = createIgnoreVerifySSL();
            Registry<ConnectionSocketFactory> socketFactoryRegistry = RegistryBuilder.<ConnectionSocketFactory>create()
                    .register("http", PlainConnectionSocketFactory.INSTANCE)
                    .register("https", new SSLConnectionSocketFactory(sslcontext))
                    .build();
            PoolingHttpClientConnectionManager connManager = new PoolingHttpClientConnectionManager(socketFactoryRegistry);
            HttpClients.custom().setConnectionManager(connManager);
            //创建自定义的httpclient对象
            CloseableHttpClient client = HttpClients.custom().setConnectionManager(connManager).build();
            //处理请求参数拼接（参数为json） 如：？name = ""&pwd = ""
            String param = changeJsonToArguments(json);
            String urlNameString = url + param;
            //创建get方式请求对象
            HttpGet get = new HttpGet(urlNameString);
            //指定报文头Content-type、User-Agent
            //get.setHeader("Content-type", "application/x-www-form-urlencoded");
            get.setHeader("User-Agent", "Mozilla/5.0 (Windows NT 6.1; rv:6.0.2) Gecko/20100101 Firefox/6.0.2");
            //执行请求操作，并拿到结果（同步阻塞）
            CloseableHttpResponse response = client.execute(get);
            //获取结果实体
            HttpEntity entity = response.getEntity();
            if (entity != null) {
                //按指定编码转换结果实体为String类型
                body = EntityUtils.toString(entity, "UTF-8");
            }
            EntityUtils.consume(entity);
            //释放链接
            response.close();
        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (KeyManagementException e) {
            e.printStackTrace();
    }finally{

        }
        return body;
    }

    public static SSLContext createIgnoreVerifySSL() throws NoSuchAlgorithmException, KeyManagementException {
        SSLContext sc = SSLContext.getInstance("SSLv3");

        // 实现一个X509TrustManager接口，用于绕过验证，不用修改里面的方法
        X509TrustManager trustManager = new X509TrustManager() {
            @Override
            public void checkClientTrusted(
                    java.security.cert.X509Certificate[] paramArrayOfX509Certificate,
                    String paramString) {
            }

            @Override
            public void checkServerTrusted(
                    java.security.cert.X509Certificate[] paramArrayOfX509Certificate,
                    String paramString) {
            }

            @Override
            public java.security.cert.X509Certificate[] getAcceptedIssuers() {
                return null;
            }
        };

        sc.init(null, new TrustManager[] { trustManager }, null);
        return sc;
    }
```

**有帮助请留言...**
<!-- 

扫码关注公众号《ipoo》
![ipoo](http://oss.ipooli.com/images/%E5%85%AC%E4%BC%97%E5%8F%B7code.jpg) -->