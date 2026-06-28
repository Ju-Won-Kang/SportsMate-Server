# SportsMate

SportsMate는 단체 스포츠에서 가장 어려운 **팀 구성, 인원 모집, 상대팀 매칭, 경기장 예약** 문제를 해결하기 위한 스포츠 매칭 플랫폼입니다. 사용자는 팀을 만들고 함께 뛸 팀원을 모집할 수 있으며, 상대 팀과 매칭하고 구장 예약 및 경기 후 평가까지 하나의 서비스 흐름에서 처리할 수 있습니다.

축구/풋살을 중심으로 농구, 야구 등 단체 스포츠에 확장 가능한 구조를 목표로 설계했습니다.

## 개발 기간

- 2024.10.20 ~ 2024.12.10
- 약 2개월

## 프로젝트 개요

| 항목 | 내용 |
| --- | --- |
| 프로젝트 유형 | 스포츠 팀 모집 및 경기장 예약/매칭 플랫폼 |
| 개발 형태 | Spring MVC 기반 WAR 배포 웹 애플리케이션 |
| 주요 사용자 | 일반 회원, 팀장, 경기장 관리자, 관리자 |
| 핵심 기능 | 회원/소셜 로그인, 팀 생성/모집, 경기장 검색/예약, 매칭, 게시판, 결제, 신고/차단 관리 |

## 기술 스택

| 구분 | 사용 기술 |
| --- | --- |
| Language | Java 8, JavaScript, HTML, CSS |
| Backend | Spring Framework 5.3.22, Spring MVC, Servlet/JSP |
| View | JSP, JSTL, jQuery, AJAX |
| Database | MariaDB |
| Persistence | MyBatis 3.5.7, mybatis-spring |
| Temporary Storage | Redis, Spring Data Redis, Lettuce |
| Security | Spring Security Crypto, BCryptPasswordEncoder, Custom Interceptor |
| External API | Naver Login, Kakao Login, KakaoPay, JavaMail |
| File Upload | Commons FileUpload, Commons IO, MultipartFile |
| Editor | Summernote |
| Logging / AOP | SLF4J, Log4j, Spring AOP, AspectJ |
| Build / Deploy | Maven, Maven Wrapper, WAR, Tomcat |
| Utility | Lombok, Gson |

## 프로젝트 아키텍처

Spring Boot가 아닌 **Spring MVC + XML 설정 기반의 전통적인 Java Web MVC 아키텍처**로 구성했습니다.

```text
Client
  ↓
JSP / jQuery / AJAX
  ↓
DispatcherServlet
  ↓
Controller
  ↓
Service
  ↓
DAO
  ↓
MyBatis Mapper
  ↓
MariaDB
```

주요 설정 파일:

- `web.xml`: Root Context 로딩, DispatcherServlet 등록, UTF-8 인코딩 필터 설정
- `servlet-context.xml`: Spring MVC 설정, JSP ViewResolver, 정적 리소스 매핑, Custom Interceptor 등록
- `root-context.xml`: DataSource, MyBatis, JavaMailSender, RedisTemplate, MultipartResolver 설정
- `spring-security.xml`: BCryptPasswordEncoder Bean 등록


## 주요 구현 포인트

### 1. Spring MVC 계층형 구조

- `Controller - Service - DAO - MyBatis Mapper` 구조로 기능별 책임을 분리했습니다.
- 회원, 팀, 경기장, 매칭, 게시판, 관리자 기능을 도메인 패키지로 나누어 관리했습니다.
- SQL은 Java 코드에 직접 작성하지 않고 기능별 MyBatis XML Mapper로 분리했습니다.

### 2. Redis 기반 이메일 인증

- 회원가입 이메일 인증번호를 Redis에 저장했습니다.
- 이메일을 key로, 6자리 인증번호를 value로 저장하며 TTL을 5분으로 설정했습니다.
- 인증번호처럼 짧게 유지되어야 하는 임시 데이터를 MariaDB에 저장하지 않아 DB 부하와 만료 데이터 관리 부담을 줄였습니다.



### 3. Custom Interceptor 기반 권한 제어

Spring Security의 URL 보안 설정보다는 프로젝트 요구사항에 맞춘 Custom Interceptor 중심으로 권한을 제어했습니다.

- `LoginInterceptor`: 로그인 여부 확인
- `TeamInterceptor`: 팀 소속 여부 확인
- `TeamLeaderInterceptor`: 팀장 권한 확인
- `TeamBoardInterceptor`: 팀 게시판 접근 권한 확인
- `TeamBoardTwoInterceptor`: 팀 게시글 작성자 권한 확인
- `BoardInterceptor`: 일반 게시글 작성자 권한 확인
- `AdminInterceptor`: 관리자 접근 권한 확인

### 4. 소셜 로그인 연동

- 네이버/카카오 OAuth 로그인 기능을 구현했습니다.
- 인가 코드로 access token을 발급받고 사용자 정보를 조회합니다.
- 기존 회원이면 로그인 처리하고, 신규 사용자면 소셜 계정 정보를 회원가입 화면으로 전달합니다.

### 5. KakaoPay 결제 연동

- 경기장 예약/매칭 결제 흐름에 KakaoPay를 연동했습니다.
- 결제 준비 API 호출 후 `tid`를 보관하고, 결제 완료 후 `pg_token`으로 승인 API를 호출합니다.
- API 응답은 DTO로 매핑해 처리했습니다.

### 6. 파일 업로드 및 에디터

- `CommonsMultipartResolver`와 `MultipartFile`을 사용해 프로필 이미지, 팀 이미지, 경기장 이미지, 게시판 첨부파일 업로드를 처리했습니다.
- 게시판 본문 작성에는 Summernote 에디터를 적용했습니다.
- Summernote 이미지 업로드는 AJAX로 서버에 파일을 전송한 뒤 본문에 이미지 URL을 삽입하는 방식으로 구현했습니다.

### 7. AOP 기반 로깅

- Spring AOP와 AspectJ를 사용해 Controller, Service, DAO 계층의 메서드 실행 흐름을 로깅했습니다.
- 반복적인 로그 코드를 비즈니스 로직에서 분리했습니다.

## 주요 기능

### 회원

- 일반 회원가입
- 경기장 관리자 회원가입
- 로그인 / 로그아웃
- 아이디 저장 쿠키
- 네이버 / 카카오 소셜 로그인
- 이메일 중복 검사
- 이메일 인증번호 발송 및 검증
- 아이디 찾기
- 임시 비밀번호 발급
- 마이페이지, 개인정보 수정, 비밀번호 변경, 회원 탈퇴

### 팀 / 구단

- 팀 생성
- 팀원 모집글 목록 / 상세 조회
- 팀 가입 신청
- 팀 가입 승인 / 거절
- 팀 정보 수정
- 팀원 관리
- 팀 탈퇴 / 팀 해체
- 팀 게시판 CRUD
- 팀 게시글 댓글 / 대댓글
- 팀 게시글 좋아요 / 신고
- 팀 투표
- 팀 매치
- 팀 랭킹

### 경기장 / 매칭

- 경기장 검색
- 경기장 상세 조회
- 경기 일정 조회
- 경기장 예약
- 매칭 신청
- 경기 결과 등록
- 경기장 리뷰
- 경기장 문의
- 환불 처리
- KakaoPay 결제

### 게시판

- 게시글 목록 / 상세 조회
- 게시글 작성 / 수정 / 삭제
- 댓글 / 대댓글
- 좋아요
- 신고
- 파일 첨부
- Summernote 이미지 업로드

### 관리자

- 신고 목록 조회
- 신고 처리
- 회원 차단 / 차단 해제
- 차단 회원 목록
- 관리자 통계 차트 조회

## 담당 및 구현 기능

- 회원가입 / 로그인 / 로그아웃
- 네이버 / 카카오 소셜 로그인 연동
- 일반 회원 및 경기장 관리자 회원가입
- BCrypt 기반 비밀번호 암호화
- 이메일 중복 검사
- JavaMail + Redis 기반 이메일 인증번호 발송/검증
- 아이디 찾기 및 임시 비밀번호 발급
- 팀 생성 및 팀원 모집 기능
- 팀 가입 신청
- 로그인 / 팀원 / 팀장 / 게시글 작성자 / 관리자 권한 Interceptor
- 공통 파일 저장 유틸
- AOP 기반 로깅


## 프로젝트 구조

```text
src/main/java/com/kh/sportsmate
├── admin        # 관리자 기능
├── board        # 일반 게시판
├── common       # 인터셉터, 메일, Redis, 소셜 API, 공통 유틸, AOP
├── match        # 매칭 및 KakaoPay 결제
├── member       # 회원가입, 로그인, 소셜 로그인
├── mypage       # 마이페이지
├── stadium      # 경기장 검색, 예약, 경기장 관리자 기능
└── team         # 팀 생성, 팀원 모집, 팀 게시판, 랭킹

src/main/resources
├── jdbc.properties      # DB 설정
├── mail.properties      # 메일 설정
├── redis.properties     # Redis 설정
├── app.properties       # 외부 API 설정
├── mybatis-config.xml   # MyBatis 설정
└── mappers              # MyBatis SQL Mapper

src/main/webapp
├── WEB-INF/spring       # Spring XML 설정
├── WEB-INF/views        # JSP 화면
└── resources            # CSS, JS, 이미지, Summernote 리소스
```

## 실행 환경

- JDK 8
- Maven 또는 Maven Wrapper
- MariaDB 11.4
- Redis 8.2
- Apache Tomcat 9.0.118 등의 Servlet Container

## 주요 URL 패턴

- 회원: `*.me`
- 마이페이지: `*.mp`
- 게시판: `*.bd`
- 팀/구단: `*.tm`
- 경기장/매칭: `*.st`, `*.gp`
- 메인 화면 AJAX: `*.mn`
- 이메일 인증: `*.mi`

## 참고 문서

- [SportsMate_요구사항_및_API명세서.xlsx](./SportsMate_요구사항_및_API명세서.xlsx)

