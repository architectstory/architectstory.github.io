---
sidebar:
  nav: "msa"
---

# Microservice Authentication

# 1. 개요

### 1) 개념

![microservice](/assets/images/authentication/authentication_define.png)
출처 https://peimsam.tistory.com/267

### 2) 분류

![microservice](/assets/images/authentication/authentication_user.png)
출처 https://peimsam.tistory.com/267

### 3) 주요특징

![microservice](/assets/images/authentication/authentication_type.png)
출처 https://peimsam.tistory.com/267

# 2. 방식

시스템에서 사용자 인증을 위한 소유기반 인증방식 중 Session(세션) 방식과 Token(토큰)방식에 대해서 알아보자.

### 1) Session 기반 인증

 ![microservice](/assets/images/authentication/authentication_session.png)
 출처 https://hackernoon.com/using-session-cookies-vs-jwt-for-authentication-sd2v3vci

 서버의 session 저장소에 SessionID가 저장
 클라이언트(사용자 브라우저)는 HTTP Cookie 헤더에 Session ID를 함께 서버로 전송

### 2) Token 기반 인증

 ![microservice](/assets/images/authentication/authentication_session.png)
 출처 https://hackernoon.com/using-session-cookies-vs-jwt-for-authentication-sd2v3vci

 인증정보를 클라이언트 직접 들고 있는 방식
 클라이언트(사용자 브라우저)의 로컬스토리지(혹은 쿠키)에 디지털서명한 토큰를 가지고 있다.
 <u>클라이언트(사용자 브라우저)는 HTTP 의 Authentication 헤더에 디지털서명이 있는 토큰을 실어 보낸다.</u>
 서버는 클라이언트에서 송신한 토큰이 위변조, 만료시간이 되었는지 확인한다.

### 3) Session vs Token
 Session은 서버에서 Session을 관리하고 클라이언트는 Session ID만 HTTP 헤더를 통해서 송신하므로 탈취시 무효처리하면된다.

 다중 서버환경에서는 여러대의 서버가 Session 을 개별 생성하고 인증할 경우, 다른 서버에서 생성한 Session 과 불일치 문제로
 인증 처리를 할 수 없다. 이러한 문제를 해결하기 위해 Sticky Session 이나 Session Clustering 방식을 사용하여 문제를
 해결한다.

 반면, <u>Token은 클라이언트(사용자 브라우저)에서 관리하므로 탈취시 위험(만료시간까지)하고, Token payload(데이타)는 암호화되어
 있지 않아 누구나 볼 수 있다.</u>

### 4) Token 을 사용하는 이유
 MSA(Micro Service Architecture) 환경에서는 Micro Service 는 독립된 서버로 동작한다.
 클라이언트 입장에서는 여러대의 서버에 접속해야 한다. Session 방식에서는 서버가 세션을 관리하기 때문에
 서버에서의 인증처리에 의존적이다. 하지만 Token 은 클라이언트가 인증정보를 가지고 있어 서버로 부터 자유롭고
 서버측에서 인증세션을 관리해야 하는 부담이 없다.

# 3. Token
### 1) Token 유형
 * 연결형: 물리적 장치가 시스템에 연결되어 액세스를 허용합니다. ex) USB 디바이스나 스마트카드를 사용
 * 비접촉형: 디바이스가 서버와 통신하려면 거리가 충분히 가까워야 하지만 연결할 필요는 없는 인증방식. ex) 매직링(마이크로소프트)
 * 분리형: 디바이스가 다른 디바이스와 접촉하지 않고도 먼 거리에서 서버와 통신와 통신. ex) MFA 휴대폰 인증


### 2) Token 인증절차

요청 -> 확인 -> 토큰 -> 저장 의 4단계로 진행

![microservice](/assets/images/authentication/authentication_okta.png)
출처 https://www.okta.com/kr/identity-101/what-is-token-based-authentication/

### 3) JSON Web Token: JWT
자바스크립트의 JSON 타입이며, Web Token 이다.
JWT 는 토큰 중 JSON 타입으로 만들어져 HTTP 프로토콜 기반으로 사용할 수 있다는 얘기다.
HTTP 는 기본적으로 state-less(무상태)
즉, 서버와 클라이언트 간의 통신 시 항상 사용자의 정보를 가지고 있지 않는 것을 지향한다.
앞서 Token 에 대한 설명에서 언급했듯이 Token 은 클라이언트에서 데이터를 관리하고,
서버가 어디든 HTTP 헤더에 Token 을 실어 보내면 된다.

![microservice](/assets/images/authentication/authentication_jwt.png)

JWT 의 구조은 Header(헤더), Payload(페이로드), Signature(시그니처)로 나뉜다.

 * Header 는 토큰 타입, 암호화 알고리즘 명시
 * Payload 는 JWT 에 넣을 데이터, JWT 발급일, JWT 만료일 등 명시
 * Signature 는  Header, Payload 가 변조 되었는지를 확인하는 역할을 한다.

### 4) Access Token 과 Refresh Token
Token 방식의 문제점은 앞서 언급했듯이 Payload 가 암호화 되지 않았고,탈취 될 수 있다.
바로 이러한 점을 보완하기 위해 적용된 매커니즘이다.
 * Access Token: 시스템에 Access 하기 위한 Token
 * Refresh Token: Access Token 을 갱신하기 위한 Token

시스템에 Access 할 수 있는 유효만료일을 Access Token 에 적용하여 만료시간 이후에는 무효화 시킨다.
하지만, 만료시간 이후에도 계속 Access Token 을 이용해서 Access 하기를 원할 경우
Refresh Token 을 이용하여 재 발급 받는다.
예를 들면 직접 API 를 호출하는 엑세스 토큰의 주기(30분)는 짧게 하고,
엑세스 토큰을 재발급하는 리프레시 토큰의 주기는 비교적 길게(1주) 한다.
엑세스 토큰이 탈취 시 시스템의 피해를 최소화 할 수 있다.

# 4. OAuth (Open Authorization)
앞서 설명한 Access Token 과 Refresh Token 을 발급하고 사용하는 프로토콜(프로세스)에 대해서 알아보자.  
OAuth 는 다양한 플랫폼에서 권한 부여(Token 방식)를 위한 산업 표준 프로토콜
타사의 사이트에 대한 접근 권한을 얻고, 그 권한을 이용하여 개발할 수 있도록 도와주는 프레임워크

OAuth 등장 이전에는 내가 만들려는 시스템에서 다른 시스템의 리소스를 가져오기 위해서는
다른 시스템에서 사용하는 Id 와 Password 를 직접 입력을 받아서(내 시스템에 저장해 두고)
필요할 때마다 Id/Password 를 이용해서 다른 시스템을 호출하였다.

이와 같은 방법은 내가 만들려는 시스템에서 다른 시스템에 등록된 Id/Password 를 받아서 관리해야하는  보안 문제가 발생한다.

예를 들면, 난 hoony.com 시스템을 만드는 사람이고, google drive를 연결해서 hoony.com에서 서비스를 하려고 한다면
hoony.com -> google drive에 접속할 수 있어야 한다.
이때 google drive에 등록된 Id/Password 를 hoony.com 시스템에 저장해 두어야 한다.
나 혼자 hoony.com를 사용하면 문제가 없겠지만, 다른 사용자들이 hoony.com을 이용하기 위해서는 hoony.com은  
각 사용자들의 google drive Id/Password를 모두 가지고 있어야 한다.

결국, <u>hoony.com -> google.com (drive) 를 사용하기 위한 인증과 특정 리소스(drive)에 Access 하기 위한 권한을
획득하는 절차가 필요하다. 바로 이러한 문제를 해결하기 위해서 인증/인가를 Access Token 으로 대체하고,
이를 발급하기 위한 일련의 과정을 인터페이스로 정의해 둔 것이 OAuth 이다.</u>

![microservice](/assets/images/authentication/authentication_oauth.png)

1. Client(클라이언트) 측에서 Resource Owner(자원 소유자)에게 인증방식 4가지 중 하나로 승인을 요청한다.
1. Resource Owner(자원 소유자)는 Client(클라기언트) 에게 권한을 부여한다.
1. Client 는 부여받은 권한으로 Authorization Server(권한 서버)에 Access Token 을 요청한다.
1. Authorization Server(권한 서버)에서 Client 와 부여 받은 권한에 대한 유효성을 검사 후 통과하면 Access Token 을 부여한다.
1. Client 는 받아온 엑세스 토큰을 이용하여 Resource Owner 의 Resource 에 접근을 요청한다.
1. Resource Server(자원 서버)는 해당 엑세스 토큰의 유효성을 검사한 후 통과하면 요청에 대한 Resource 를 Client 에 넘겨준다.  

자세한 내용은 아래의 링크 사이트 참고 

[OAuth2.0](https://inpa.tistory.com/entry/WEB-%F0%9F%93%9A-OAuth-20-%EA%B0%9C%EB%85%90-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC){:target="_blank"}

# 4. OAuth (Open Authorization)