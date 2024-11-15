# 24-25 Server-Assignment-06
<img width="974" alt="리프레시토큰다이어그램" src="https://github.com/user-attachments/assets/3f691fc4-b9b9-48d7-b377-57a8fc56f832">

## Description
- JWT를 활용한 회원가입
- JWT로 로그인
- 로그인한 사용자의 댓글 포스팅 (Create)
- 로그인한 사용자의 모든 댓글 목록 읽기 (Read)
- 로그인한 사용자의 본인 댓글 수정 (Update)
- 로그인한 사용자의 본인 댓글 삭제 (Delete)
- JWT 로그아웃
- RefreshToken을 이용한 AccessToken을 다시 발급받는 기능을 포함

## Important content

- 로그인시 액세스 토큰과 리프레시 토큰을 같이 주는 로직이 괜찮은건지 궁금합니다
- 접근권한에 대한 처리 로직이 적절한지 궁금합니다.

## Question
RToken : 86400000ms (1일) 리프레쉬 토큰  /  AToken : 1800000ms (30분) 액세스 토큰
- 1일과 30분은 각각의 토큰에 적절한 유효시간인건지 궁금합니다.

- JWT 토큰 방식이 무상태서버를 만드는데 효과적이고, 서버에 부하가 적지만
만료전까지의 통제가 어렵고 토큰 탈취시 보안 이슈가 발생된다는 점을 학습하였습니다.
이를 보완하는 방법으로써 어떤 방법들이 더 있는지도 궁금합니다. 
(찾아본 것으로는 블랙리스트 활용, HttpOnly 쿠키 사용 등이 있었습니다.)

## 리프레쉬 토큰이란 무엇이고 왜 필요할까?

우리는 jwt기반 인증을 구현하면서 서버와 클라이언트가 인증하는 수단으로 “토큰”을 통해 인증하고 서비스를 사용할 수 있게 규정해주었습니다. 
그런데 만약 이 토큰을 주고받는 환경의 보안이 취약해
공격자가 토큰을 탈취하는 일이 발생하면 어떻게 될까요?

만약 여러분이 콘서트 티켓을 예매했는데 티켓의 정보가 
노출되어 공격자가 이 티켓으로 콘서트를 보려고 한다고 가정해봅시다.
토큰은 이미 발급되면 그 자체로 인증수단이 되므로 서버에서는 그 토큰과 함께 들어온 요청이 공격자의 요청인지 확인할 수 없습니다. 

그래서 고안해낸 방법이 토큰의 만료기한을 짧게 설정하는 것이었지요? 그런데 유효기간이 너무 짧다면 사용자가 이용하는 도중에 토큰이 만료될 수도 있다는 문제가 발생합니다.

이러한 문제를 해결하기 위해 리프레쉬 토큰이 해결책이 될 수 있습니다.

리프레쉬 토큰은 액세스 토큰과 별개로서 
사용자 인증을 위한것이 아니라 
액세스 토큰이 만료되었을 때, 
이를 다시 발급하기 위해서 사용합니다.

액세스 토큰의 유효기간은 짧게, 리프레쉬 토큰의 유효기간은 길게 설정한다면 
공격자가 액세스 토큰을 탈취하더라도 몇 분 뒤에는 무력화되니 더 안전하겠지요?

## 그렇다면 어떤 순서로 JWT인증이 이루어지는가?
- 회원가입
/auth/signup
→ 사용자 정보를 데이터베이스에 저장
- 로그인
/auth/login
→ AccessToken, RefreshToken 발급
- Access Token 갱신
/auth/refresh
→ Refresh Token을 받아서 Access Token 재발행
- 로그아웃
/auth/logout
→ Refresh Token을 삭제

- 댓글 CRUD API
/comments (댓글 등록)
/comments/all (모든 댓글 조회)
/comments/{id} (댓글 수정/삭제)
→ JwtFilter에서 유효한 access토큰인지 인증 후 컨트롤러 처리

## 요점 정리
AccessToken은 클라이언트단에 저장되고 (쿠키, 세션스토리지) 사용자가 서버의 
api를 요청할 때마다 reqbody 또는 헤더로 받아서 서버에서 인증과 인가 수행
접근 권한이 없거나 틀린 토큰임을 확인하면 요청 거부
만료된 토큰으로 요청시 Refresh토큰을 통해 AccessToken 재발행
별도의 세션 관리가 없으므로 서버는 상태를 유지하지 않음. (무상태) 
그러므로 서버에 걸리는 부하는 적다는 장점이 있음.

하지만 단점으로 
토큰 만료 전까지의 통제가 어렵고 
토큰의 길이로 인해 네트워크에 부담이 가중됨, 
클라이언트의 쿠키나 로컬스토리지가 탈취당할 가능성 존재.

## Reference
스프링부트3 백엔드 개발자 되기 서적
[어떤-로그인-방식을-선택해야-할까 벨로그 방예혁](https://velog.io/@hyeok_1212/%EC%96%B4%EB%96%A4-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EB%B0%A9%EC%8B%9D%EC%9D%84-%EC%84%A0%ED%83%9D%ED%95%B4%EC%95%BC-%ED%95%A0%EA%B9%8C)
[GDGoC-JWT를-활용한-로그인-구현 벨로그 김우진](https://velog.io/@woogym/GDGoC-JWT%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EA%B5%AC%ED%98%84)

<img width="724" alt="스크린샷 2024-11-15 오후 9 14 31" src="https://github.com/user-attachments/assets/ff85a985-f0f5-4ba1-8d7a-55f7eb09ab1c">
<img width="893" alt="스크린샷 2024-11-15 오후 9 15 04" src="https://github.com/user-attachments/assets/8e91022b-5b93-49b1-8f7a-75d4c3c8b2c2">
<img width="891" alt="스크린샷 2024-11-15 오후 9 15 16" src="https://github.com/user-attachments/assets/2d4e2aac-1fc1-49b8-b77e-d946f670b07c">
<img width="883" alt="스크린샷 2024-11-15 오후 9 15 37" src="https://github.com/user-attachments/assets/ba8b2134-c32a-40bc-a8bc-8870f9e1a1f9">
<img width="887" alt="스크린샷 2024-11-15 오후 9 16 13" src="https://github.com/user-attachments/assets/9ec6e616-45c0-4f3e-9a4a-2f2b017e0f88">
<img width="881" alt="스크린샷 2024-11-15 오후 9 15 54" src="https://github.com/user-attachments/assets/51c8b185-c77e-46d1-9721-09a3e0829111">
<img width="650" alt="스크린샷 2024-11-15 오후 9 16 40" src="https://github.com/user-attachments/assets/ad48cd68-ca9b-4e20-bed9-f7db6a2fa394">
<img width="741" alt="스크린샷 2024-11-15 오후 9 18 01" src="https://github.com/user-attachments/assets/2191eab5-0475-4c6e-94be-eaf6788b6c68">
