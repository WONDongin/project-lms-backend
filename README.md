### ⭐ Learning Management System (학사 관리 시스템)

<br>

### 소개 (Overview)

> JSP Model 2(MVC) 아키텍처를 기반으로 구현된 학사 관리 시스템(LMS)입니다.
교수, 학생, 관리자 권한에 따라 수업, 강의, 회원, 게시판 등을 관리할 수 있는 웹 애플리케이션입니다.

<Br>

### ⚙️ 주요 기능 (Features)

- 학생 
  - 강의 목록/과제 조회
  - 수강 신청/취소, 내 정보 확인
  - 게시글 수정/작성

- 교수
  - 강의 개설 / 수정
  - 학생 목록 조회
  - 과제 업로드 및 점수 부여

- 관리자
  - 사용자 전체 관리 (CRUD)
  - 교수/학생 계정 승인
  - 학과/강의/과목 관리

- 인증 & 권한
  - 세션 기반 로그인
  - 관리자/교수/학생 권한 분리
  - 접근 제어

<Br>

### ⚙️ 기술 스택 (Tech Stack)
| 구분                    | 사용 기술                                          |
| --------------------- |------------------------------------------------|
| **Language**          | Java 17 (LTS)                                  |
| **Backend Framework** | JSP, Servlet (Model 2 MVC)                     |
| **DB Layer**          | MyBatis (Mapper Interface + XML 기반), JDBC      |
| **Database**          | MySQL / MariaDB                                |
| **Web / View**        | JSP, JSTL, EL, HTML5, CSS3, JavaScript         |
| **Architecture**      | MVC(Model 2), DAO 패턴, Service Layer, Mapper 구조 |
| **Session/Auth**      | HttpSession 기반 로그인 및 권한(Role) 제어               |
| **Build/Deploy**      | WAR 배포 구조, Apache Tomcat 9                     |
| **Tools**             | GitHub Desktop, Eclipse, HeidiSQL              |
 
<Br>

### 📂 프로젝트 구조 (Project Structure)

```bash
/project-lms-backend
 ├── src/main/java
 │     ├── controllers/           # Servlet Controller
 │     ├── models/
 │     │     ├── users/           # UserDAO, UserDTO
 │     │     ├── boards/          # 게시판 관련 DAO
 │     │     ├── others/          # 강의/학과 등
 │     │     └── mappers/         # MyBatis Mapper
 │     └── MyBatisConnection.java
 │
 ├── src/main/webapp
 │     ├── views/                 # JSP 화면
 │     └── WEB-INF/
 │         ├── lib/               # 라이브러리(JAR)
 │         └── web.xml            # 서블릿 설정

```

<br>

### 📂 전체 아키텍처 (Architecture)
- 전통적인 `JSP Model2(MVC)` 구조
- 역할 분리(`Controller`, `View`, `DAO`) 명확
- `MyBatis`로 `SQL` 관리 분리
세션 기반 인증 + 권한 분기

```bash
[Client] 
   ↓
[JSP View]  ← JSTL, EL
   ↓
[Controller (Servlet)] 
   ↓
[Service / Business Logic] 
   ↓
[DAO] 
   ↓
[MyBatis Mapper (Interface + XML)] 
   ↓
[MySQL Database]
```

<br>

### 🗄 핵심 로직 (Core Logic)

<br>

### 1. 전체 요청 흐름 (MVC2 기반 Request Flow)

사용자의 모든 요청은 아래와 같은 순서로 처리됩니다.
- Filter 단계에서 인코딩과 로그인/권한을 1차로 제어
- `Controller`에서 요청 URL에 따라 기능 분배
- `Service`/`DAO`/`MyBatis` 계층에서 실제 데이터 처리
- JSP `View`에서 최종 화면 렌더링

```bash
[Client (Browser)]
   ↓  HTTP 요청 (GET/POST)
[Filter]
   - CharEncodingFilter : UTF-8 인코딩 설정
   - UriFilter          : 로그인/권한 체크 및 접근 제어
   ↓
[Controller (Servlet, *LMSController / UserController 등)]
   ↓
[Service / Business Logic]
   ↓
[DAO]
   ↓
[MyBatis Mapper (Interface + XML / Annotation)]
   ↓
[MySQL / MariaDB]
   ↑
[JSP View 렌더링 (JSTL, EL)]
```

<br>

### 2. 인증 & 권한 관리 (Session + Role 기반 Access Control)
#### 2-1. 로그인 흐름
1. `loginForm.jsp` 에서 아이디/비밀번호 입력
2. `UserController`의 /`users`/`login` 매핑으로 요청 전달 
3. `UserDao` → `UserMapper.selectOne(user_no)` 로 사용자 조회
4. 비밀번호 일치 시:
   - `HttpSession에` `login`(User 객체) 저장
   - 사용자 `role`(관리자/교수/학생)에 따라 메인 페이지/메뉴 분기
5. 실패 시:
- 메시지(msg)와 이동 경로(url)를 설정 후 alert.jsp로 이동

> 세션에 로그인 정보가 없거나, 권한이 부족할 경우
> UriFilter에서 사전에 차단하고 로그인 페이지 또는 에러 페이지로 이동시킵니다.

<br>

#### 2-2. 권한(Role)에 따른 접근 제어
- 권한은 세션에 저장된 User 정보의 role 값으로 판단하며,
  특정 컨트롤러/기능은 해당 `Role`만 접근 가능하게 설계

>`ADMIN` : 전체 사용자 관리, 학과/강의/과목 등록 및 수정
> 
>`PROFESSOR` : 강의 개설, 과제 등록, 성적 관리
> 
>`STUDENT` : 수강 신청/취소, 과제 제출, 내 정보/성적 조회



<br>

### 3. 도메인별 핵심 흐름
#### 3-1. 수강 및 강의 관리 (Class / Reg_class / ClassLMS)
- 수강 취소, 성적 조회 등도 동일한 패턴으로
- `Controller` → `Service/DAO` → `Mapper` → `DB` → `JSP` 흐름

대표 흐름 예시: 수강 신청

```bash
코드 복사
수강 신청 버튼 클릭 (JSP)
   ↓
ClassLMSController (요청 파라미터: 학정번호, 학번 등)
   ↓
Service/DAO에서 수강 가능 여부 체크
   - 정원 초과 여부
   - 중복 신청 여부
   - 수강 가능 학년/학과 여부
   ↓
Reg_class 테이블 Insert (수강 신청 정보 저장)
   ↓
결과 메시지 및 수강 목록 화면 반환
```

<br>

#### 3-2. 학과/게시판/공지 관리 (DeptLMS / Board)
- 교수/학생 권한에 따라 작성/수정/삭제 가능 여부가 달라집니다.
- 과제 게시판, 공지사항, Q&A 등도 동일한 구조로 확장

대표 흐름 예시: 게시글 목록 조회 & 상세보기

```bash
학과 게시판 메뉴 클릭
   ↓
DeptLMSController / BoardController
   ↓
검색 조건(학과, 제목, 작성자 등) 구성
   ↓
BoardDao → BoardMapper (페이징 + 조건 검색)
   ↓
게시글 목록 JSP에 데이터 바인딩 후 렌더링
   ↓
상세보기 클릭 시, 게시글 번호로 재조회 후 상세 페이지 이동
```

<br>

### 4. MyBatis 기반 DB 접근 구조
#### 4-1. 공통 DB 연결 (MyBatisConnection)
> 모든 DAO가 동일한 방식으로 DB에 접근하도록 통일

- SqlSessionFactory를 애플리케이션 시작 시 1회만 생성
- DAO에서는 MyBatisConnection.getConnection() 으로 SqlSession을 받아 사용
- 작업 후 MyBatisConnection.close(session)에서 commit + close 처리

<br>

#### 4-2. DAO & Mapper 패턴
- DAO는 비즈니스에 가까운 메서드 이름 사용 (예: findUserById, insertClass, selectBoardList
- Mapper 인터페이스 + SQL(XML/Annotation)을 통해 자바 코드와 SQL을 분리하여 유지보수성을 높였습니다.
```bash
Controller
   ↓
DAO (UserDao, BoardDao, ClassDao 등)
   ↓
MyBatis Mapper (UserMapper, BoardMapper …)
   ↓
SQL (XML / Annotation)
```
<br>

### 5. 예외 처리 & 확장 고려 사항
- 로그인/수강 신청/게시글 작성 등 주요 기능에서 실패 시 메시지 + 이동 URL을 공통 JSP(alert.jsp)에서 처리
- `Filter`와 `Controller` 레벨에서 잘못된 접근을 사전에 방지
- `DAO/MyBatis` 계층을 통해 모듈화되어 있어, 추후 `Spring MVC` / `Spring Boot + JPA` 구조로 리팩토링이 가능하도록 설계했습니다.


<Br>

### 🔍 화면 예시 (Screenshots)

#### MainLMS (메인 페이지 / 공지사항)
<img src="./docs/screenshots/mainLms.png" />
<img src="./docs/screenshots/notion.png" />

#### DeptLMS (학과 게시판 / 강의관리)
<img src="./docs/screenshots/deptLMS.png" />

#### ClassLMS (과제/학점 관리)
<img src="./docs/screenshots/classLMS.png" />

<Br>

### 📄 배운 점 (What I Learned)

- JSP/Servlet 기반 전통적 MVC 구조 실무식 구현
- MyBatis 기반 DAO/Mapper 구조로 DB 접근 계층 설계
- 세션(Session) 기반 인증·권한 체계 처리
- 프로젝트 전반에 걸친 데이터 흐름 구조 이해
- 웹 요청/응답 흐름(HttpServletRequest/Response) 숙련
- Git을 통한 버전 관리 및 협업 준비

  <Br>

### 📄 추후 개선 계획 (Improvements)

- Spring Boot 기반 구조로 리팩토링
- JPA 도입하여 DAO 계층 단순화
- REST API 기반 구조로 확장
- JWT 기반 인증 적용
- 프론트엔드(Vue/React) 분리형 구조로 발전
