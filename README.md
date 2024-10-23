## Introduction

⏳ 개발 기간: 2024.05 ~ 2024.06

👨🏻‍💻 프로젝트 소개
  - 프론트부터 백엔드 담당 네이버클라우드 우분투 서버를 통해 배포.
  - 사용자가 위치, 날씨, 선호도를 바탕으로 음식과 식당을 추천해주고, 이용한 장소들에 대해 내용을 공유할 수 있는 웹 어플리케이션.

## Architecture
![스크린샷 2024-10-22 오후 12 55 57](https://github.com/user-attachments/assets/55c30c96-8801-48c0-abbc-e17aa6b0a52f)

## ERD
![스크린샷 2024-10-22 오후 3 10 24](https://github.com/user-attachments/assets/a7ed74f7-dc40-4412-aa75-afa971370213)

<details>
  <summary>📂 서비스 주요 기능</summary>

  ### Member
    - 홈페이지 회원가입을 통한 로그인
    - sns 로그인
    - 회원 정보 수정

  ### Recommend
    - 음식 선호도 설문조사
    - 사용자가 위치한 곳의 날씨 정보 불러오기
    - 사용자가 선호할 만한 음식과 식당 추천

  ### Board
    - 컨텐츠 '좋아요' 하기
    - 컨텐츠 '좋아요' 취소
    - 컨텐츠 작성
    - 사용자 위치 컨텐츠 불러오기
</details>

## Description

🔎 목차
  - [MSA 구축](#msa-구축)
  - [RDB 설계 이유](#rdb-설계-이유)
  - [Spring security를 적용한 OAuth2 & JWT 기반 로그인 기능](#spring-security를-적용한-oauth2--jwt-기반-로그인-기능)
  - [Mybatis 사용](#mybatis-사용)
  - [MongoDB의 사용 이유](#mongodb-사용-이유)
  - [화면 렌더링](#화면-렌더링)
  - [배포](#배포)
  - [코드 설계](#코드-설계)

---

## MSA 구축

두 개의 독립적인 프로젝트를 MSA로 구성하였습니다.
프로젝트 규모상 모놀리식 아키텍처로 개발하는 것이 더 유리하지만, 학습이 가장 큰 목표이기 때문에 MSA로 개발을 하였습니다.

<details>
<summary>1. MSA(Mycroservice Architecture) vs MA(Monolithic Architecture)</summary>
  
### MSA의 장점
- 각 서비스가 독립적으로 배포되고 확장이 가능하여 확장과 유지보수가 뛰어납니다.
- 일부 서비스가 다운되더라도 전체 다른 서비스에 영향을 미치지 않습니다.
- 독립적인 여러 서비스가 서로 다른 기술, 프레임워크를 사용할 수 있어 새로운 기술 도입에 용이합니다.
  
### MSA의 단점
- 서비스간 통신하기 위해 복잡한 통신과정과 데이터 관리가 필요합니다.
- 여러 서비스가 독립적으로 배포되기 때문에 CI/CD 파이프라인 관리등 배포와 관리 비용이 높아집니다.
- 서비스간 통신으로 인해 네트워크 지연의 성능 저하가 발생할 수 있습ㅂ니다.

### MA의 장점
  - 초기 개발에 유리하며 빠르게 개발이 가능합니다.
  - 복잡한 통신 과정이 필요 없습니다.
  - 하나의 프로세스 내에서 모든 컴포넌트가 실행이 되기 때문에 규모가 작은 프로젝트의 경우 성능이 더 뛰어납니다.

### MA의 단점
- 프로젝트 규모가 커질수록 복잡해지고 유지관리, 확장이 어려워 집니다
- 특정 기능만 수정사항이 발생해도 전체 어플리케이션을 배포해야 하기 때문에 확장에 비용이 증가합니다.
- MSA와 달리 특정 기술, 프레임워크에 종속하게 되어 새로운 기술 도입이 어려울 수 있습니다.
  
</details>

<details>
  
<summary>2. MSA 환경</summary>  

  ### API Gateway
  ** Spring cloud gateway VS Spring cloud Zuul**
  - Zuul의 패치가 중단되기도 하였고, 비동기 방식 기반으로 요청을 빠르게 처리할 수 있는 Spring Cloud Gateway를 적용하였습니다.
  - blocking server와 다르게 비동기 방식을 기반으로 하나의 스레드당 여러 요청을 수행할 수 있어 대용랑 트래픽에 유리합니다.
  - 수많은 요청을 큐에 넣어 적합한 서비스로 라우팅해주는 역할을 빠르게 수행할 수 있습니다.
    
  ** 역할**
  - 모든 클라이언트의 요청을 받아 적절한 마이크로서비스로 연결해 줍니다.
  - 각각의 마이크로서비스들은 서로의 포트번호를 몰라도 됩니다.
  - 클라이언트는 Gateway 포트만 알면 됩니다.
  - 여러 마이크로서비스에 대한 API를 통합하여 클라이언트가 여러 서비스에 접근하지 않아도 하나의 진입점으로 얻을 수 있습니다.
  
</details>

</br>

## RDB 설계 이유

** MEMBER
- oAuth2 사용자와 홈페이지 사용자의 로그인 기능을 구현하기 위해서 단순히 하나의 테이블로 이들의 정보를 저장할 수는 없었습니다.
- oAuth 사용자는 pw가 존재하지 않기 때문에 회원을 구분할 수 있는 최소 조건인 uid, nickname을 필드로 갖는 MEMBER 테이블과 회원 상세정보를 의미하는 MEMBER_DETAIL 테이블을 자식으로 가집니다.
- 부모 테이블로는 oAuth 사용자인지, 홈페이지 사용자인지를 구분할 수 있는 IDP, 권환을 의미하는 ROLE 테이블을 갖습니다.

** 그외
- 의미적으로 관련있는 속성들끼리만 테이블을 구성하도록 하였습니다.
- 중복 데이터를 최대한 허용하지 않도록 설계하였습니다.
- Join을 수행했을때 가짜 데이터가 생기지 않도록 설계하였습니다.
- 되도록 null 값을 가지지 않도록 설계하였습니다.

</br>

## Spring security를 적용한 OAuth2 & JWT 기반 로그인 기능

**1. 로그인 기능 구현**
- Spring security를 적용하여 OAuth2와 JWT 기반의 로그인 기능을 구현했습니다.
- Spring Security는 filter 흐름에 따라 인증과 권한에 대한 부분을 처리하며, 일일이 보안관련 로직을 작성하지 않아도 되기에 적용하였습니다.
- OAuth2 도입 이유는 간편 로그인을 통해 사용자 정보를 직접 관리할 필요가 없어지고, 사용자 입장에서 보다 편리하게 로그인을 할 수 있기 때문에 도입하게 되었습니다.

**2. 세션과 JWT 비교**
- 세션 방식과 JWT 방식의 장단점을 비교하여 JWT 기반으로 결정하게 됐습니다.
- 세션 기반은 Stateful 한 방식으로 서버에서 사용자 정보를 저장해야 하므로 확장성에 제한이 있습니다. Redis를 활용하여 세션 클러스터링을 구현할 수 있지만, Redis는 이미 캐시 저장소로 사용되고 있으므로 추가적인 리소스 비용 발생의 부담이 생깁니다. 그리고 많은 사용자가 접속할 경우 서버의 부하가 증가한다는 단점이 있습니다.
- 반면에, JWT 기반 방식은 Stateless 한 방식으로 서버는 사용자의 상태를 저장하지 않기 때문에 확장성이 높아집니다. 추후 분산 서버 환경을 구축할 때 JWT 방식이 세션 클러스터링을 하지 않고 서버를 빠르게 확장해나갈 수 있다고 판단했습니다.
- 또한 세션은 탈취 당하면 보안에 문제가 생길 수 있지만, JWT는 자체적으로 정보를 암호화할 수 있어 데이터의 무결성을 보장할 수 있습니다.
- 물론, JWT 방식도 완전히 안전한 것은 아닙니다. 클라이언트 측에서 토큰을 안전하게 보관해야 하며, 매 요청마다 토큰을 보내야 하므로 헤더의 크기가 커지는 단점도 있습니다.
- 그리고 토큰 만료시 새로운 토큰을 발급해줘야 하는 과정도 필요하게 됩니다.

**3. JWT 채택**
- 결과적으로, 세션과 토큰 방식의 장단점을 비교했을 때, MSA 구축을 위해 JWT 방식을 선택하게 됐습니다.
- UsernamePasswordAuthenticationFilter를 재정의 하여 JWT 안중 필요한 자원의 사용을 가능하게 하였습니다.

</br>

## 화면 렌더링

**1. Vue.js**
- ㅁㄴㅇㄹㅁㄴㅇㄹㅁㄴㅇㄹ

