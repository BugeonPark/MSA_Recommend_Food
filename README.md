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
  - [Spring security를 적용한 OAuth2 & JWT 기반 로그인 기능](#spring-security를-적용한-oauth2--jwt-기반-로그인-기능)
  - [N+1 문제 해결](#n1-문제-해결)

---

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


