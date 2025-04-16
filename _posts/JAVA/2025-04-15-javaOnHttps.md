---
layout: post
title: '[JAVA] 자바로 HTTPS 요청 보내기 '
subtitle: 'HTTPS, SSL, peer not authenticated, HttpClient'
date: 2025-04-15 12:00:00 +0900
categories: 'JAVA'
background: '/img/posts/javaetc/java.png'
---

# 자바7, Apache HttpClient 4.2.5 에서 HTTPS 요청을 보낼 떄

- HTTPS 요청을 보낼 때, 자바는 내부적으로 cacerts 파일을 이용해 서버 인증서를 검증한다.
- 그런데 사설 인증서나 신뢰할 수 없는 CA의 인증서를 쓰는 서버에 요청할 경우, javax.net.ssl.SSLPeerUnverifiedException: peer not authenticated 오류가 난다.
- 아래 코드는 신뢰할 수 있는 인증서를 cacerts에 등록했을 때, 그 인증서를 사용하도록 HttpClient를 설정해주는 코드이다.

<br>

## 먼저 caerts에 인증서를 넣어보자

1.접속할 사이트에 들어가서 자물쇠를 클릭해 인증서 보기로 모든 체인을 확인해 한 단계씩 모두 인증서를 추출한다.

- 주소창 옆 자물쇠 > 보안 or 이 연결은 안전합니다. > 인증서 or 인증서가 유효함 > 세부정보 > 인증서 계층 > 단계별 순서대로 [내보내기]

<br>

2.인증서를 병합한다.

- window 기준

```bash
copy a.crt + b.crt resultChain.crt
```

- mac 기준

```bash
cat a.crt b.crt > resultChain.crt
```

<br>

3.인증서를 등록한다. 

- window 기준

```bash
keytool -import -alias kifin_chain -keystore "%JAVA_HOME%\jre\lib\security\cacerts" -file resultChain.crt
```

- mac 기준

```bash
sudo keytool -import \
  -alias kifin_cert \
  -keystore "$JAVA_HOME/jre/lib/security/cacerts" \
  -file ~/Downloads/resultChain.crt
```

- 기본 비번은 changeit 이다. 

<br>

4.인증서를 잘 들어갔는지 확인한다.

- window, mac 둘 다 같은 명령어로 확인 가능
- %JAVA_HOME% 위치 잘 맞춰서 적어주면 된다. 

```bash
keytool -list -keystore "%JAVA_HOME%\jre\lib\security\cacerts" -alias kifin_cert -storepass changeit
```

<br>

# JAVA 에서 HTTPS 요청 날리기

```java
public class SSLClient {

    public static void main(String[] args) throws Exception {

        // 결과값을 위한 변수들
        BufferedReader reader = new BufferedReader();
        String line = null;
        StringBuffer result = new StringBuffer();

        // 1. cacerts 경로와 비밀번호
        String cacertsPath = System.getProperty("java.home") + "/lib/security/cacerts";
        String cacertsPassword = "changeit";

        // 2. Java Key Store 로드
        KeyStore trustStore = KeyStore.getInstance(KeyStore.getDefaultType());
        FileInputStream instream = new FileInputStream(cacertsPath);
        trustStore.load(instream, cacertsPassword.toCharArray());
        instream.close(); // 스트림 닫기

        // 3. TrustManagerFactory 생성
        TrustManagerFactory tmf = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
        tmf.init(trustStore);

        // 4. SSLContext 생성 및 초기화    
        SSLContext sslContext = SSLContext.getInstance("TLSv1.2");
        sslContext.init(null, tmf.getTrustManagers(), new SecureRandom());

        // 5. SSLSocketFactory 상수, 기본값: SSLSocketFactory. STRICT_HOSTNAME_VERIFIER
        SSLSocketFactory sf = new SSLSocketFactory(sslContext, SSLSocketFactory.STRICT_HOSTNAME_VERIFIER);

        // 6. SchemeRegistry 구성
        SchemeRegistry sr = new SchemeRegistry();
        sr.register(new Scheme("https", 443, sf));

        // 7. ConnectionManager 구성
        PoolingClientConnectionManager pccm = new PoolingClientConnectionManager(sr);
        pccm.setMaxTotal(100); // Connection Pool 최대 사이즈
        pccm.setDefaultMaxPerRoute(20); // 각 host(IP와 Port의 조합)당 Connection Pool에 생성가능한 Connection 수
        
        // 8. DefaultHttpClient 생성 및 요청
        HttpClient httpClient = new DefaultHttpClient(pccm);

        HttpGet httpUrl = new HttpGet("https://yourWebSite?param1=value1");
        HttpResponse response = httpClient.execute(httpUrl);
        HttpEntity entity = response.getEntity();

        // 응답 바디의 값을 꺼내 보여줍니다. 
        if(entity != null) {
            reader = new BufferedReader(new InputStreamReader(entity.getContent(), "UTF-8"));
            while((line = reader.readLine()) != null) {
                result.append(line);
            }
            reader.close();
        }

        httpClient.getConnectionManager().shutdown();

        // 결과 출력 
        System.out.println("Result : " + result.toString());
    }
}
```

## 1. cacerts 경로와 비밀번호
- 보통 자바 폴더 아래 /lib/security/cacerts 에 위치하고 있습니다.
- cacerts는 자바 기본 키스토어로, 신뢰할 수 있는 루트 인증서들을 담고 있습니다.
- 경로는 자바 홈 디렉토리 기준이고, 기본 비밀번호는 "changeit" 이며 변경이 가능합니다.

<br>

## 2. Java Key Store 로드
- KeyStore 인스턴스를 생성하고, cacerts 파일을 열어서 인증서를 메모리에 로딩합니다.
- 이걸 SSLSocketFactory에 넘겨서, HTTP 클라이언트가 사용할 수 있도록 합니다.

<br>

## 3. TrustManagerFactory 생성

- KeyStore (신뢰할 CA 목록)를 기반으로 TrustManager 객체 배열을 만들어줍니다.
- 운영 환경에서 CA 서명된 인증서를 사용하는 서버에 연결할 때 필수입니다.

<br>

## 4. SSLContext 생성

- SSLSocketFactory에 넘기기 위한 SSLContext를 생성합니다.
> - TLS 1.2 이상 강제 가능합니다. 
> > - TLS(Transport Layer Security): SSL(Secure Sockets Layer)와 혼동되어 사용되는 용어입니다. SSL은 넷스케이프사에서 1995년에 개발한 HTTP 암호화 프로토콜입니다. 하지만 SSL에는 여러 보안 취약점이 있었기 때문에 사용이 중단되었고, 더 안전한 TLS 프로토콜로 대체되었습니다.
> > - 최신 버전은 TLS 1.3이며 보안을 위해 TLS 1.2 이상 사용을 권고합니다. 
> - 커스텀 TrustManager, KeyManager 사용 가능합니다.
> - 클라이언트 인증서 설정도 가능합니다.

- SSLContext를 생성하지 않는 간단한 방법도 있습니다. 
> - 간단한 방식은 SSLContext.getInstance("TLS")를 기본적으로 호출합니다.

```java
SSLSocketFactory socketFactory = new SSLSocketFactory(trustStore);
```

<br>

## 5. SSLSocketFactory 상수

- setHostnameVerifier() 를 이용해서 호스트 네임 검증 수준를 정할 수 있습니다.

**ALLOW_ALL_HOSTNAME_VERIFIER**

```java
SSLSocketFactory sf = new SSLSocketFactory(sslContext, SLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER);
```

- 인증서의 CN과 도메인 이름이 불일치해도 허용합니다.
- 개발용, 테스트용에서만 사용해야 합니다. 

<br>

**BROWSER_COMPATIBLE_HOSTNAME_VERIFIER**

```java
SSLSocketFactory sf = new SSLSocketFactory(sslContext, SLSocketFactory.BROWSER_COMPATIBLE_HOSTNAME_VERIFIER);
```

- 대부분의 브라우저처럼 느슨한 CN 검증합니다.
- CN과 SAN 둘 다 허용합니다.
> - CN : Common Name, 일반이름
> - SAN : Subject Alternative Name, 주체 대체 이름

<br>

**STRICT_HOSTNAME_VERIFIER**

```java
SSLSocketFactory sf = new SSLSocketFactory(sslContext, SLSocketFactory.STRICT_HOSTNAME_VERIFIER);
```

> - 가장 엄격한 방식이고, 디폴트 값입니다.
> - CN과 정확히 일치해야 하고, SAN을 허용하지 않습니다.

<br>

## 6. SchemeRegistry 구성

- HttpClient 4.xx 버전은 프로토콜별 연결 방식(http, https)을 SchemeRegistry에 등록해줘야 합니다. 
- 여기서는 HTTPS 요청시 위에서 만든 socketFactory를 사용하도록 등록했습니다. 
> - 스키마 : 프로토콜(ex. http, https)과 해당 프로토콜을 사용하기 위한 소켓 팩토리, 포트 번호 등

<br>

## 7. ConnectionManager 구성

```java
ClientConnectionManager ccm = new SingleClientConnManager(sr);
```

- ClientConnectionManager 기존 방식은 deprecated 되었습니다.
- connection reuse가 안 되고,멀티스레드 지원되지 않습니다. 

<br>

**PoolingClientConnectionManager**

```java
PoolingClientConnectionManager pccm = new PoolingClientConnectionManager(sr);
```

- 여러 연결 재사용 가능하고, thread-safe하게 변경되었습니다. 

<br>

## 8. DefaultHttpClient 생성 및 요청

- DefaultHttpClient 객체를 생성해서 요청을 날려 응답값을 확인합니다.
- 참고: HttpGet 요청을 보낼 때 한글이 있을 경우 아래 처럼 인코딩 과정이 필수입니다.

```java
String param1 = URLEncoder.encode("한글", "UTF-8");
```

<br>

- 오래된 버전이므로 자바7 이상의 버전을 사용하시는 분은 다른 곳에서 참고하시는 것이 좋을 듯합니다.
- Java 7은 오래된 버전이라 많은 최신 인증기관이 빠져 있어 가능하시다면 자바 업그레이드를 고려해 보안 수준을 높이는 방법이 좋을 듯 합니다.

<br>

Reference : 
- [Apache HttpClient를 사용하여 Https 요청하기](https://jinwooe.blogspot.com/2013/12/apache-httpclient-https.html)