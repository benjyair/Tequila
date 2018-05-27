---
title: Android使用指定APN发起网络请求
layout: post
date: 2017/06/21 14:30:39
tags : Android
---

### 如果想在不切换现有 APN 的情况下让一个网络请求走另一个 APN 应该怎么做？

这样的需求在 Android 上是可行的，你只需要获取到一个 Network 对象就可以。
Network 提供 openConnection() 方法，你可以像使用 HttpURLConnection 一样来发起网络请求。
```java
/**
 * /frameworks/base/core/java/android/net/Network.java
 * Opens the specified {@link URL} on this {@code Network}, such that all traffic will be sent
 * on this Network. The URL protocol must be {@code HTTP} or {@code HTTPS}.
 *
 * @param proxy the proxy through which the connection will be established.
 * @return a {@code URLConnection} to the resource referred to by this URL.
 * @throws MalformedURLException if the URL protocol is not HTTP or HTTPS.
 * @throws IllegalArgumentException if the argument proxy is null.
 * @throws IOException if an error occurs while opening the connection.
 * @see java.net.URL#openConnection()
 */
public URLConnection openConnection(URL url, java.net.Proxy proxy) throws IOException {
   if (proxy == null) throw new IllegalArgumentException("proxy is null");
   maybeInitHttpClient();
   String protocol = url.getProtocol();
   OkUrlFactory okUrlFactory;
   // TODO: HttpHandler creates OkUrlFactory instances that share the default ResponseCache.
   // Could this cause unexpected behavior?
   if (protocol.equals("http")) {
       okUrlFactory = HttpHandler.createHttpOkUrlFactory(proxy);
   } else if (protocol.equals("https")) {
       okUrlFactory = HttpsHandler.createHttpsOkUrlFactory(proxy);
   } else {
       // OkHttp only supports HTTP and HTTPS and returns a null URLStreamHandler if
       // passed another protocol.
       throw new MalformedURLException("Invalid URL or unrecognized protocol " + protocol);
   }
   OkHttpClient client = okUrlFactory.client();
   client.setSocketFactory(getSocketFactory()).setConnectionPool(mConnectionPool);
   // Let network traffic go via mDns
   client.setDns(mDns);
   return okUrlFactory.open(url);
}
```
以上方法中的 Proxy 对象便是由 APN 中设置的 Proxy 和 ProxyPort 等参数生成的，使用方法参考如下例子：
```java
/**
 * /packages/services/Mms/src/com/android/mms/service/MmsHttpClient.java
 * Execute an MMS HTTP request, either a POST (sending) or a GET (downloading)
 *
 * @param urlString  The request URL, for sending it is usually the MMSC, and for downloading
 *                   it is the message URL
 * @param pdu        For POST (sending) only, the PDU to send
 * @param method     HTTP method, POST for sending and GET for downloading
 * @param isProxySet Is there a proxy for the MMSC
 * @param proxyHost  The proxy host
 * @param proxyPort  The proxy port
 * @param mmsConfig  The MMS config to use
 * @param subId      The subscription ID used to get line number, etc.
 * @param requestId  The request ID for logging
 * @return The HTTP response body
 * @throws MmsHttpException For any failures
 */
public byte[] execute(String urlString, byte[] pdu, String method, boolean isProxySet,
                      String proxyHost, int proxyPort, Bundle mmsConfig, int subId, String requestId)
        throws MmsHttpException {
    LogUtil.d(requestId, "HTTP: " + method + " " + redactUrlForNonVerbose(urlString)
            + (isProxySet ? (", proxy=" + proxyHost + ":" + proxyPort) : "")
            + ", PDU size=" + (pdu != null ? pdu.length : 0));
    checkMethod(method);
    HttpURLConnection connection = null;
    try {
        Proxy proxy = Proxy.NO_PROXY;
        if (isProxySet) {
            proxy = new Proxy(Proxy.Type.HTTP,
                    new InetSocketAddress(mNetwork.getByName(proxyHost), proxyPort));
        }
        final URL url = new URL(urlString);
        maybeWaitForIpv4(requestId, url);
        // Now get the connection
        connection = (HttpURLConnection) mNetwork.openConnection(url, proxy);
        connection.setDoInput(true);
        connection.setConnectTimeout(
                mmsConfig.getInt(SmsManager.MMS_CONFIG_HTTP_SOCKET_TIMEOUT));
        // ------- COMMON HEADERS ---------
        // Header: Accept
        connection.setRequestProperty(HEADER_ACCEPT, HEADER_VALUE_ACCEPT);
        // Header: Accept-Language
        connection.setRequestProperty(
                HEADER_ACCEPT_LANGUAGE, getCurrentAcceptLanguage(Locale.getDefault()));
        // Header: User-Agent
        final String userAgent = mmsConfig.getString(SmsManager.MMS_CONFIG_USER_AGENT);
        LogUtil.i(requestId, "HTTP: User-Agent=" + userAgent);
        connection.setRequestProperty(HEADER_USER_AGENT, userAgent);
        // Header: x-wap-profile
        final String uaProfUrlTagName =
                mmsConfig.getString(SmsManager.MMS_CONFIG_UA_PROF_TAG_NAME);
        final String uaProfUrl = mmsConfig.getString(SmsManager.MMS_CONFIG_UA_PROF_URL);
        if (uaProfUrl != null) {
            LogUtil.i(requestId, "HTTP: UaProfUrl=" + uaProfUrl);
            connection.setRequestProperty(uaProfUrlTagName, uaProfUrl);
        }
        // Header: Connection: close (if needed)
        // Some carriers require that the HTTP connection's socket is closed
        // after an MMS request/response is complete. In these cases keep alive
        // is disabled. See https://tools.ietf.org/html/rfc7230#section-6.6
        if (mmsConfig.getBoolean(SmsManager.MMS_CONFIG_CLOSE_CONNECTION, false)) {
            LogUtil.i(requestId, "HTTP: Connection close after request");
            connection.setRequestProperty(HEADER_CONNECTION, HEADER_CONNECTION_CLOSE);
        }
        // Add extra headers specified by mms_config.xml's httpparams
        addExtraHeaders(connection, mmsConfig, subId);
        // Different stuff for GET and POST
        if (METHOD_POST.equals(method)) {
            if (pdu == null || pdu.length < 1) {
                LogUtil.e(requestId, "HTTP: empty pdu");
                throw new MmsHttpException(0/*statusCode*/, "Sending empty PDU");
            }
            connection.setDoOutput(true);
            connection.setRequestMethod(METHOD_POST);
            if (mmsConfig.getBoolean(SmsManager.MMS_CONFIG_SUPPORT_HTTP_CHARSET_HEADER)) {
                connection.setRequestProperty(HEADER_CONTENT_TYPE,
                        HEADER_VALUE_CONTENT_TYPE_WITH_CHARSET);
            } else {
                connection.setRequestProperty(HEADER_CONTENT_TYPE,
                        HEADER_VALUE_CONTENT_TYPE_WITHOUT_CHARSET);
            }
            if (LogUtil.isLoggable(Log.VERBOSE)) {
                logHttpHeaders(connection.getRequestProperties(), requestId);
            }
            connection.setFixedLengthStreamingMode(pdu.length);
            // Sending request body
            final OutputStream out =
                    new BufferedOutputStream(connection.getOutputStream());
            out.write(pdu);
            out.flush();
            out.close();
        } else if (METHOD_GET.equals(method)) {
            if (LogUtil.isLoggable(Log.VERBOSE)) {
                logHttpHeaders(connection.getRequestProperties(), requestId);
            }
            connection.setRequestMethod(METHOD_GET);
        }
        // Get response
        final int responseCode = connection.getResponseCode();
        final String responseMessage = connection.getResponseMessage();
        LogUtil.d(requestId, "HTTP: " + responseCode + " " + responseMessage);
        if (LogUtil.isLoggable(Log.VERBOSE)) {
            logHttpHeaders(connection.getHeaderFields(), requestId);
        }
        if (responseCode / 100 != 2) {
            throw new MmsHttpException(responseCode, responseMessage);
        }
        final InputStream in = new BufferedInputStream(connection.getInputStream());
        final ByteArrayOutputStream byteOut = new ByteArrayOutputStream();
        final byte[] buf = new byte[4096];
        int count = 0;
        while ((count = in.read(buf)) > 0) {
            byteOut.write(buf, 0, count);
        }
        in.close();
        final byte[] responseBody = byteOut.toByteArray();
        LogUtil.d(requestId, "HTTP: response size="
                + (responseBody != null ? responseBody.length : 0));
        return responseBody;
    } catch (MalformedURLException e) {
        final String redactedUrl = redactUrlForNonVerbose(urlString);
        LogUtil.e(requestId, "HTTP: invalid URL " + redactedUrl, e);
        throw new MmsHttpException(0/*statusCode*/, "Invalid URL " + redactedUrl, e);
    } catch (ProtocolException e) {
        final String redactedUrl = redactUrlForNonVerbose(urlString);
        LogUtil.e(requestId, "HTTP: invalid URL protocol " + redactedUrl, e);
        throw new MmsHttpException(0/*statusCode*/, "Invalid URL protocol " + redactedUrl, e);
    } catch (IOException e) {
        LogUtil.e(requestId, "HTTP: IO failure", e);
        throw new MmsHttpException(0/*statusCode*/, e);
    } finally {
        if (connection != null) {
            connection.disconnect();
        }
    }
}
```

### 参考
[/frameworks/base/core/java/android/net/Network.java](http://androidxref.com/8.0.0_r4/xref/frameworks/base/core/java/android/net/Network.java)
[/packages/services/Mms/src/com/android/mms/service/MmsHttpClient.java](http://androidxref.com/8.0.0_r4/xref/packages/services/Mms/src/com/android/mms/service/MmsHttpClient.java)
[/packages/services/Mms/src/com/android/mms/service/MmsNetworkManager.java](http://androidxref.com/8.0.0_r4/xref/packages/services/Mms/src/com/android/mms/service/MmsRequest.java)
<br/>
