#### HTTP/1.0부터 HTTP/3까지 차근차근 알아보자

# 2.5.1 HTTP/1.0

![](https://velog.velcdn.com/images/noahshin__11/post/51c82338-310f-47a5-8fb1-c9765992a40d/image.png)

- 서버로부터 파일을 가져올 때마다 TCP의 3-웨이 핸드셰이크를 계속해서 열어야 하기 때 문에 RTT가 증가하는 단점이 있었습니다.

> — RTT (Round-Trip Time)
패킷이 목적지에 도달하고 나서 다시 출발지로 돌아오기까지 걸리는 시간이며 패킷 왕복 시간

### RTT의 증가를 해결하기 위한 방법

- 매번 연결할 때마다 RTT가 증가하니 서버에 부담이 많이 가고 사용자 응답 시간이 길어져.
- 이를 해결하기 위해 이미지 스플리팅, 코드 압축, 이미지 Base64 인코딩을 사용 하곤 했다.

> - **이미지 스플리팅**
많은 이미지를 다운로드받게 되면 과부하가 걸리기 때문에 많은 이미지가 합쳐 있는 하나 의 이미지를 다운로드받고, 이를 기반으로 background-image의 position을 이용하여 이 미지를 표기하는 방법 입니다. (코드는 봐도 모르니 생략)

- **코드 압축**
코드 압축은 코드를 압축해서 개행 문자, 빈칸을 없애서 코드의 크기를 최소화하는 방법 입니다.

```
const express = require('express')
const app = express()
const port = 3000

app.get(, (req, res) => { 
res.send('Hello World!')
})

app.listen(port, () => {
console.log('Example app listening on port ${port}')
})
```

위를 아래처럼...

```
const express=require("express"),app=express()zport=3e3;app. get("/"z(ezp)=>{p.send("Hello World!")}),app.listen(3e3z()=>{console. log("Example app listening on port 3000")});
```

이렇게 개행 문자, 띄어쓰기 등이 사라져 코드가 압축되면 코드 용량이 줄어듭니다.
((이게 의미가 있나..? 그럼 ts -> js 빌드할때도 이렇게 하면 안되나?))

> - 이미지 Base64 인코딩
이미지 파일을 64진법으로 이루어진 문자열로 인코딩하는 방법.
이 방법을 사용하면 서버와의 연결을 열고 이미지에 대해 서버에 HTTP 요청을 할 필요가 없다는 장점이 있다.
하지만 Base64 문자열로 변환할 경우 37% 정도 크기가 더 커지는 단점이 있다.

# 2.5.2 HTTP/1.1

**HTTP/1.0에서 발전한 것이 바로 HTTP/1.1
**

매번 TCP 연결을 하는 것이 아니라 한 번 TCP 초기화를 한 이후에 keep-alive라는 옵션으로 여러 개의 파일을 송수신할 수 있게 바뀌었다.
(참고로 HTTP/1.0에서도 keep-alive가 있었지만 표준화가 되어 있지 않았고 HTTP/1.1 부터 표준화가 되어 기본 옵션으로 설정됐다.)
![](https://velog.velcdn.com/images/noahshin__11/post/0ecdcdd5-1816-40b9-b061-2c8fefaa7446/image.png)

- 다음 그림처럼 한 번 TCP 3-웨이 핸드셰이크가 발생하면 그다음부터 발생하지 않는 것 을 볼 수 있다.
- 하지만 문서 안에 포함된 다수의 리소스(이미지, css 파일, script 파일) 를 처리하려면 요청할 리소스 개수에 비례해서 대기 시간이 길어지는 단점이 있다.
(파일들이 크거나 많으면 한번의 GET-RESPONSE 의 시간이 길어지는 것)

### HOL Blocking

> HOL Blocking(Head Of Line Blocking)은 네트워크에서 같은 큐에 있는 패킷이 그 첫 번 째 패킷에 의해 지연될 때 발생하는 성능 저하 현상을 말합니다.

![](https://velog.velcdn.com/images/noahshin__11/post/ae426e5d-1df7-4357-82fc-978b9ac352d5/image.png)
> 예를 들어 앞의 그림처 럼 image.jpg와 styles.css, data.xml을 다운로드받을 때 보통은 순차 적으로 잘 받아지지만 image.jpg가 느리게 받아진다면 그 뒤에 있는 것들이 대기하게 되 며 다운로드가 지연되는 상태가 되는 것

### 무거운 헤더 구조

HTTP/1.1 의 헤더에는 쿠키 등 많은 메타데이터가 들어 있고 압축이 되지 않아 무거웠다. 그래서 !

# 2.5.3 HTTP/2

- HTTP/2는 SPDY 프로토콜에서 파생된 HTTP/l.x보다 지연 시간을 줄이고 응답 시간을 더 빠르게 할 수 있으며 멀티플렉싱, 헤더 압축, 서버 푸시, 요청의 우선순위 처리를 지원 하는 프로토콜.

## 멀티플렉싱

- 멀티플렉싱이란 여러 개의 스트림을 사용하여 송수신한다는 것.
이를 통해 특정 스트림의 패킷이 손실되었다고 하더라도 해당 스트림에만 영향을 미치고 나머지 스트림은 멀쩡하게 동작할 수 있습니다.

> - 스트림 (stream)
시간이 지남에 따라 사용할 수 있게 되는 일련의 데이터 요소를 가리키는 데이터 흐름
![](https://velog.velcdn.com/images/noahshin__11/post/deb40614-ef80-40b8-993f-ea0c0619ee4c/image.png)
앞의 그림은 하나의 연결 내 여러 스트림을 캡처한 모습. 병렬적인 스트림(stream) 들을 통해 데이터를 서빙하고 있다. 또한, 스트림 내의 데이터들도 쪼개져 있죠. 애플리케이션에서 받아온 메시지를 독립된 프레임으로 조각내어 서로 송수신한 이후 다시 조립하며 데이터를 주고 받는다.
![](https://velog.velcdn.com/images/noahshin__11/post/a1975581-debe-49ed-94ff-49a66bd94f16/image.png)
 이를 통해 단일 연결을 사용하여 병렬로 여러 요청을 받을 수 있고 응답을 줄 수 있다. 이렇게 되면 HTTP/1.x에서 발생하는 문제인 HOL Blocking을 해결할 수 있다.

## 헤더 압축

HTTP/l.x에는 크기가 큰 헤더라는 문제가 있었다.
![](https://velog.velcdn.com/images/noahshin__11/post/febabc99-a235-4c18-80f2-970dc6c70d5a/image.png)
이를 HTTP/2에서는 헤더 압축을 써서 해결하는데, 허프만 코딩 압축 알고리즘을 사용하는 HPACK 압축 형식을 가진다.

> 허프만 코딩
허프만 코딩(huffman coding)은 문자열을 문자 단위로 쪼개 빈도수를 세어 빈도가 높은 정보는 적은 비트 수를 사용하여 표현하고, 빈도가 낮은 정보는 비트 수를 많이 사용하여 표현해서 전체 데이터의 표현에 필요한 비트양을 줄이는 원리입니다.

## 서버 푸시

HTTP/1.1 에서는 클라이언트가 서버에 요청을 해야 파일을 다운로드받을 수 있었다면,
HTTP/2는 클라이언트 요청 없이 서버가 바로 리소스를 푸시할 수 있다.
![](https://velog.velcdn.com/images/noahshin__11/post/373ec324-836e-4d98-a755-d8b34cae5657/image.png)
html에는 css나 js 파일이 포함되기 마련인데 html을 읽으면서 그 안에 들어 있던 css 파일 을 서버에서 푸시하여 클라이언트에 먼저 줄 수 있다.

# 2.5.4 HTTPS

- **HTTP/2는 HTTPS 위에서 동작**.
- HTTPS는 애플리케이션 계층과 전송 계층 사이에 신뢰 계층인 SSL/TLS 계층을 넣은 신뢰할 수 있는 HTTP 요청을 말한다.
- 이를 통해 ‘**통신을 암호화**’합니다.

## SSL/TLS

- SSL(Secure Socket Layer)은 SSL 1.0부터 시작해서 SSL 2.0, SSL 3.0, TLS(Transport Layer Security Protocol) 1.0, TLS 1.3까지 버전이 올라가며 마지막으로 TLS로 명칭이 변경되었으나,
보통 이를 합쳐 SSL/TLS로 많이 부릅니다.
이 책에서는 최신 TLS 버전인 TLS 1.3을 기반으로 설명.
SSL/TLS은 전송 계층에서 보안을 제공하는 프로토콜.
클라이언트와 서버가 통신 할 때 SSL/TLS를 통해 제3자가 메시지를 도청하거나 변조하지 못하도록 한다.

![](https://velog.velcdn.com/images/noahshin__11/post/ac061060-bdac-4df3-8545-eaf8828e445d/image.png)

### 보안 세션

보안이 시작되고 끝나는 동안 유지되는 세션을 말하고, SSL/TLS는 핸드셰 이크를 통해 보안 세션을 생성하고 이를 기반으로 상태 정보 등을 공유합니다.

> 세션 : 운영체제가 어떠한 사용자로부터 자신의 자산 이용을 허락하는 일정한 기간을 뜻한다. 즉, 사용자는 일정 시간 동안 응용 프로그램, 자원 등을 사용할 수 있다.
![](https://velog.velcdn.com/images/noahshin__11/post/a0f2b4a2-438b-43c5-a9fd-b1efb5e8b5d5/image.png)

- 클라이언트와 서버와 키를 공유하고 이를 기반으로 인증, 인증 확인 등의 작업이 일어나는 단 한번의 1-RTT가 생긴 후 데이터를 송수신하는것을 볼 수 있다.
- 클라이언트에서 사이퍼 슈트(cypher suites)를 서버에 전달하면 서버는 받은 사이퍼 슈트 의 암호화 알고리즘 리스트를 제공할 수 있는지 확인.
- 제공할 수 있다면 서버에서 클라이언트로 인증서를 보내는 인증 메커니즘이 시작되고 이후 해싱 알고리즘 등으로 암호화된 데이터의 송수신이 시작됩니다.

#### 사이퍼 슈트

![](https://velog.velcdn.com/images/noahshin__11/post/bd341f4e-996c-4739-b76e-ed958cc09ce0/image.png)
(넘어가) 예를 들어 TLS_AES_128_GCM_SHA256에는 세 가지 규약이 들어 있는데 TLS는 프로 토콜, AES_128_GCM은 AEAD 사이퍼 모드, SHA256은 해싱 알고리즘을 뜻합니다.

#### 인증 메커니즘

- 인증 메커니즘은 CA(Certificate Authorities)에서 발급한 인증서를 기반으로 이루어진다.
- CA에서 발급한 인증서는 안전한 연결을 시작하는 데 있어 필요한 ‘**공개키**’를 클라이언트에 제공하고 사용자가 접속한 ‘**서버가 신뢰**’할 수 있는 서버임을 보장합니다.
- 인증서는 **서비스 정보. 공개키, 지문, 디지털 서명** 등으로 이루어져 있습니다.
- 참고로 CA는 아무 기업이나 할 수 있는 것이 아니고 신뢰성이 엄격하게 공인된 기업들만 참여할 수 있으며. 대표적인 기업으로는 Comodo, GoDaddy, GlobalSign, 아마존 등이 있습니다.

### CA 발급 과정

- 자신의 서비스가 CA 인증서를 발급받으려면 자신의 사이트 정보와 공개키를 CA에 제출 해야 합니다.
- 이후 CA는 공개키를 해시한 값인 지문(finger print)을 사용하는 CA의 비밀 키 등을 기반으로 CA 인증서를 발급합니다.
![](https://velog.velcdn.com/images/noahshin__11/post/dd1f1025-36a1-44c0-ad2b-90bfd45aa3be/image.png)

### 암호화 알고리즘

키 교환 암호화 알고리즘으로는 대수곡선 기반의 ECDHE(Elliptic Curve Diffie-Hellman Ephermeral) 또는 모듈식 기반의 DHE(DifFie—Hellman Ephermeral)를 사용합니다. 둘 다 디피-헬만(Diffie-Hellman) 방식을 근간으로 만들어졌습니다.

((이거까지 알아야하나))

#### 디피-헬만 키 교환 암호화 알고리즘

디피-헬만 키 교환(Diffie—Hellman key exchange) 암호화 알고리즘은 암호키를 교환하는 하나의 방법입니다.

![](https://velog.velcdn.com/images/noahshin__11/post/5ae64548-c640-4277-892f-f7b8282d6673/image.png)

앞의 식에서 g와 x와 p를 안다면 y는 구하기 쉽지만 g와 y와 p만 안다면 x를 구하기는 어렵다는 원리에 기반한 알고리즘.
![](https://velog.velcdn.com/images/noahshin__11/post/8b5da676-19a7-41fa-b9fd-c6df186bfc2f/image.png)
1도 모르겠다.

### 해싱 알고리즘

해싱 알고리즘은 데이터를 추정하기 힘든 더 작고, 섞여 있는 조각으로 만드는 알고리즘 입니다. SSL/TLS는 해싱 알고리즘으로 SHA-256 알고리즘과 SHA-384 알고리즘을 쓰며,

그중 많이 쓰는 SHA-256 알고리즘을 설명.

## SHA-256 알고리즘

SHA-256 알고리즘은 해시 함수의 결괏값이 256비트인 알고리즘이며 비트 코인을 비롯 한 많은 블록체인 시스템에서도 씁니다. SHA-256 알고리즘은 해싱을 해야 할 메시지에
1을 추가하는 등 전처리를 하고 전처리된 메시지를 기반으로 해시를 반환합니다.
![](https://velog.velcdn.com/images/noahshin__11/post/b8580b29-74ee-4343-a70e-732d09274acf/image.png)
![](https://velog.velcdn.com/images/noahshin__11/post/3dea0a87-6c86-49d4-952a-0ca5da163b3d/image.png)
> ⚠️ 참고로 TLS 1.3은 사용자가 이전에 방문한 사이트로 다시 방문한다면 SSL/TLS에서 보안 세션을 만들 때 걸리는 통신을 하지 않아도 됩니다. 이를 0-RTT라고 합니다.

## SEO에도 도움이 되는 HTTPS

구글(Google)은 SSL 인증서를 강조해왔고 사이트 내 모든 요소가 동일하다면 HTTPS 서비스를 하는 사이트가 그렇지 않은 사이트보다 SEO 순위가 높을 것이라고 공식적으로 밝혔다.

SEO (Search Engine Optimization)는 검색엔진 최적화를 뜻하며
사용자들이 구글, 네이버 같은 검색엔진으로 웹 사이트를 검색했을 때 그 결과를 페이지 상단에 노출시켜 많은 사람이 볼 수 있도록 최적화하는 방법을 의미.
서비스를 운영한다면 SEO 관리는 필수입니다.
내가 만든 사이트에 많은 사람이 유입되면 좋겠죠?
이를 위한 방법으로 캐노니 컬 설정, 메타 설정, 페이지 속도 개선, 사이트맵 관리 등이 있습니다.

((약간 프론트의 냄새가 나는데...그러니까 이미지 캡쳐))
![](https://velog.velcdn.com/images/noahshin__11/post/21698daf-e5f1-45b1-8d34-e2aa050c84fd/image.png)![](https://velog.velcdn.com/images/noahshin__11/post/59b91059-ebc8-4ef2-b493-e88f420bf63f/image.png)![](https://velog.velcdn.com/images/noahshin__11/post/a1ab0747-a444-4940-9497-4e20aaaaf423/image.png)

## HTTPS 구축방법

HTTPS 구축 방법은 크게 세 가지

1. 직접 CA에서 구매한 인증키를 기반으로 HTTPS 서비스를 구축하거나,
2. 서버 앞단의 HTTPS를 제공하는 로드밸런서를 두거나,
3. 서버 앞단에 HTTPS를 제공하는 CDN을 둬서 구축합니다.



# 2.5.5 HTTP/3

- HTTP/3은 HTTP/1.1 및 HTTP/2와 함께 World Wide Web에서 정보를 교환하는 데 사용되는 HTTP의 세 번째 버전.
- TCP 위에서 돌아가는 HTTP/2와는 달리 HTTP/3은 QUIC이라는 계층 위에서 돌아가며, TCP 기반이 아닌 UDP 기반으로 돌아 갑니다.
  ![](https://velog.velcdn.com/images/noahshin__11/post/cf2f45fd-8744-400c-94fe-e9fdcbdc5a56/image.png)

또한, HTTP/2에서 장점이었던 멀티플렉싱을 가지고 있으며 초기 연결 설정 시 지연 시 간 감소라는장점이 있습니다.

### 초기 연결설정 시 지연 시간 감소

QUIC은 TCP를 사용하지 않기 때문에 통신을 시작할 때 번거로운 3-웨이 핸드셰이크 과정을 거치지 않아도 된다.


![](https://velog.velcdn.com/images/noahshin__11/post/2fd6785d-c4a7-4034-9b3d-2368b2ba3bd3/image.png)
- QUIC은 첫 연결 설정에 1-RTT만 소요.
- 클라이언트가 서버에 어떤 신호를 한 번 주고, 서버도 거기에 응답하기만 하면 바로 본 통신을 시작할 수 있다는 것.
- 참고로 QUIC은 순방향 오류 수정 메커니즘(FEC, Forword Error Correction)이 적용되 었다. 
- 이는 전송한 패킷이 손실되었다면 수신 측에서 에러를 검출하고 수정하는 방식 이며 열악한 네트워크 환경에서도 낮은 패킷 손실률을 자랑합니다.

![](https://velog.velcdn.com/images/noahshin__11/post/bac076b6-64f4-4519-afb6-5afc21e856c7/image.png)
d


















