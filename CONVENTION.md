# Moy-Log BackEnd Server Convention

> 이 문서는 Moy-Log 프로젝트의 기술 스택, 모듈 구조, 코딩 컨벤션을 정의합니다.

---

# 1. Project Convention

## 기본 프로젝트 세팅

| 항목              | 버전         | 비고                     |
|:----------------|:-----------|:-----------------------|
| **Java**        | 25 (LTS)   | 최신 가상 스레드 및 언어 기능 활용   |
| **Spring Boot** | 4.0.5      | Spring Framework 7 기반  |
| **Gradle**      | Groovy DSL | `buildSrc` 패턴으로 컨벤션 관리 |

- **언어 및 프레임워크**: 최신 LTS 버전 기반 안정성 및 성능 최적화
- **빌드 시스템**: `buildSrc` 기반 Custom Convention Plugin 사용
  - `java-convention`: 순수 Java 모듈용 (Jacoco 80% 커버리지 규칙 포함)
  - `spring-convention`: Spring Boot 모듈용 (Jacoco 50% 커버리지 규칙 포함)
  - `DependencyVersions.groovy`: 모든 라이브러리 버전 중앙 관리
---

## 모듈 구조 및 패키지 설계

### 📂 전체 모듈 구조
```text
Github-Repository/
├── moylog-app/                    # [Spring] 실행 가능한 애플리케이션 (Entry Point)
├── moylog-common/                 # [Pure Java] 비즈니스 무관 공통 유틸
├── moylog-domain/                 # [Pure Java] 핵심 비즈니스 로직 및 인터페이스(Port)
├── moylog-external/               # [Spring] 외부 인프라 구현체 (Adapter)
└── buildSrc/                      # [Gradle] 공통 빌드 설정 및 버전 관리
```

### 📦 모듈별 상세 패키지

> **1. app (Application & Presentation)**

* 성격: 시스템의 진입점이며 유즈케이스를 조립함.
* 주요 패키지
  * `com.moylog.app.<feature>.presentation`: 진입점별 컨트롤러 분리
    * `.api`: REST API 전용 (Mobile/Web)
    * `.chat`: WebSocket, STOMP 메시지 핸들러
    * `.batch`: 스케줄러 및 배치 작업 핸들러
  * `com.moylog.app.<feature>.application`: 애플리케이션 로직 (@Transactional 등 관리)
  * `com.moylog.app.<feature>.dto`: 요청/응답용 DTO
  * `com.moylog.app.auth`: Security 설정, JWT 관련 클래스
  * `com.moylog.app.global`: app 모듈 전역 설정 - 예외, 응답 등

**⚠️중요**: security의 경우 framework 의존적이므로 app 모듈에 위치한다. 

> **2. domain (Core Domain)**

* 성격: 외부 기술에 의존하지 않는 순수 비즈니스 로직.
* 주요 패키지
  * `com.moylog.domain.<feature>.model`: 엔티티, VO (Value Object)
  * `com.moylog.domain.<feature>.repository`: Repository 인터페이스 (Port)
  * `com.moylog.domain.<feature>.exception`: 도메인 전용 예외
  * `com.moylog.domain.shared`: 공용 VO 등

> **3. external (Infrastructure)**

* 성격: 도메인과 무관하게 교체 가능한 외부 시스템 연동.
* 주요 패키지
  * `com.moylog.external.redis`: 캐싱 및 분산 락 구현
  * `com.moylog.external.clients`: 외부 Open API 연동 (Feign, WebClient)
  * `com.moylog.external.persistence.<featrue>`: JPA/QueryDSL 등 레포지토리 구현체

**⚠️중요**: persistence의 경우 쿼리 튜닝으로 인해 빈번한 수정이 발생한다. app 모듈의 독립성을 위해 external에 위치한다.

> **4. common (Shared Kernel)**
   
* 성격: 전역에서 사용되는 독립적인 유틸리티.
* 주요 패키지
  * `com.moylog.common.utils`: 날짜, 문자열 등 범용 유틸
  * `com.moylog.common.exception`: BusinessException, ErrorCode 정의
  * `com.moylog.common.annotation`: @Logging, @LoginUser 등 정의
---

## 모듈 의존성 흐름

```
app → domain, common, external (runtime)
external → domain, common
domain → common
```

* 원칙 1: domain과 common은 Spring 의존성 없이 Pure Java를 유지한다.
* 원칙 2: 의존성은 항상 안쪽(domain)으로 향한다.
* 원칙 3: app의 **Application Logic**은 외부 구현체(external)에 직접 의존하지 않으며, 도메인이 정의한 인터페이스를 통해서만 외부 세계와 소통한다. <br> 구현체와의 결합은 오직 **설정 계층(Configuration)** 에서만 일어난다.


---

## buildSrc 구조

```
buildSrc/
├── src/main/groovy/
│   ├── java-convention.gradle       # 순수 Java 모듈 공통 설정
│   ├── spring-convention.gradle     # Spring 모듈 공통 설정
│   └── DependencyVersions.groovy    # 버전 중앙 관리
└── build.gradle
```

### 👉 DependencyVersions.groovy 예시
```groovy
class DependencyVersions {
    static final String SPRING_BOOT = '4.0.5'
    static final String QUERYDSL    = '5.1.0'
    static final String P6SPY       = '3.9.1'
}
```

---

# 2. Code Convention

## 자바 코딩 규칙

* var 키워드 사용 금지: 명시적인 타입을 선언하여 타입 안정성과 코드 가독성을 최우선으로 한다.
* 불변성 유지: 모든 필드는 가능한 final을 사용하며, Setter 대신 의미 있는 이름의 메서드를 사용한다.
* boilerplate code를 제거하기 위해 Lombok 사용은 허용한다.

---

## 예외 처리 전략

> **기본 구조: BaseException (RuntimeException 상속)을 최상위로 하며, 아래 예외들이 이를 상속하여 비즈니스 예외를 세분화한다.**

* Common
  * DomainException: 비즈니스 예외 (HTTP STATUS: 400)
  * EntityNotFoundException: 리소스를 찾을 수 없을 때 (HTTP STATUS: 404)
  * DuplicateApplicationException: 중복된 데이터가 존재할 때 (HTTP STATUS: 409)
* External
  * BadGateWayException: 외부 시스템(타 서버)에서 잘못된 응답을 받았을 때 (HTTP STATUS: 502)
  * ServiceUnavailableException: 외부 서비스 과부하 (HTTP STATUS: 503)
* 그 외 추가적인 예외는 이후 필요에 따라 추가한다.

### 스택 트레이스

비즈니스 예외는 트레이스를 생성하지 않는다.  
성능을 최적화하고 5XX 예외만 로그에 트레이스를 남긴다.

### 예외 코드 예시

```java
public class BaseException extends RuntimeException {

  private final ErrorCode errorCode;

  public BaseException(ErrorCode errorCode) {
    super(errorCode.mesage());
    this.errorCode = errorCode;
  }

  public BaseException(ErrorCode errorCode, Object... args) {
    super(errorCode.mesage(args));
  }

  public ErrorCode getErrorCode() {
    return errorCode;
  }
}
```

```java
public interface ErrorCode {
    String name();                  // 에러 코드 식별자 (예: "MEMBER_NOT_FOUND")
    String message();               // 기본 에러 메시지
    String message(Object... args); // 가변 인자가 포함된 에러 메시지 (String.format 활용)
}
```

> 가변 인자를 지원하여 메시지를 동적으로 구성할 수 있도록 설계한다.

---

## API 응답 구조

* **RESTful Status Code**: 성공(200, 201), 실패(4xx, 5xx) 등 의미에 맞는 HTTP 상태 코드를 명확히 사용한다.
* **ResponseEntity 활용**: 모든 컨트롤러 메서드는 일관성과 유연성을 위해 `ResponseEntity`를 반환한다.
* **표준 래퍼**: 모든 응답 바디는 `data`, `error`, `timestamp` 구조를 유지한다.
* **필드 규칙**
  * 성공 시: `data`에 결과 담기, `error`는 `null`
  * 실패 시: `data`는 `null`, `error`에 상세 정보 담기
* **확장성**: 단일 값(ID 등)을 반환하더라도 반드시 객체 형태(`{"id": 1}`)로 감싸서 `data`에 담는다.

### 성공 응답 (2XX)
```json
{
  "data": { "id": 1 },
  "error": null,
  "timestamp": "2026-04-25 18:05:14"
}
```

### 에러 응답 (4xx, 5xx)
```json
{
  "data": null,
  "error": {
    "code": "MEMBER_NOT_FOUND",
    "message": "회원을 찾을 수 없습니다.",
    "detail": null
  },
  "timestamp": "2026-04-25 18:06:01"
}
```

--

## 클래스 네이밍 규칙

### 프로덕션 클래스
> Controller: \<Feature>Controller
* 예: PostController, MemberController

> Service: \<Feature>Service
* 예: PostService, AuthService

> Repository  
* 인터페이스 (Domain): <Entity>Repository  
* 구현체 (App/Persistence): Jpa<Entity>Repository, Querydsl<Entity>Repository

> DTO (Data Transfer Object)
* Suffix 필수: 요청은 Request, 응답은 Response를 접미사로 붙인다.  
* 예: PostCreateRequest, MemberInfoResponse
* Inner Class 금지: 각 DTO는 별도의 파일로 관리하여 재사용성과 가독성을 높인다.

> Domain Model: 비즈니스 의미를 담은 명사형으로 작성한다.  
* 예: Post, Author, Money

### 테스트 클래스

> 클래스명: \<Target>Test
  * 예: PostServiceTest, PostRepositoryTest
  * 메서드명: CamelCase를 사용하며, 반드시 영어로 작성한다. (Jacoco 리포트 및 CI 환경의 가독성을 위함)
    * 예: createPost_Success, findMember_Fail_WhenNotFound
  * 설명: @DisplayName을 통해 한글로 비즈니스 요구사항을 명시한다.


### Exception

> \<Reason>Exception  
* 예: PostNotFoundException, InvalidTokenException

---

## 테스트 코드 컨벤션

### 테스트 프레임워크
- **JUnit5** 통일 (AssertJ 사용)
- Mockito 사용 (모킹 필요 시)

### 테스트 네이밍 규칙
- 메서드명은 영어, 한글 설명은 `@DisplayName`에 작성

```java
@Test
@DisplayName("정상적인 요청이면 게시글이 생성된다")
void createPost() {
    // given
    // when
    // then
}
```

### 통합 테스트 베이스 클래스: `IntegrationTest`
- 통합 테스트는 반드시 **`IntegrationTest`를 상속**해서 작성

```java
// api/src/test/java/com/moylog/api/support/IntegrationTest.java
@SpringBootTest
@ActiveProfiles("test")
@Transactional
public abstract class IntegrationTest {
}
```

### 통합 테스트 작성법
```java
@DisplayName("게시글 서비스 통합 테스트")
class PostServiceTest extends IntegrationTest {

    @Autowired
    private PostService postService;

    @Test
    @DisplayName("정상적인 요청이면 게시글이 생성된다")
    void createPost() {
        // given
        PostCreateRequest request = PostFixture.createRequest();

        // when
        PostResponse response = postService.create(1L, request);

        // then
        assertThat(response.title()).isEqualTo(request.title());
    }
}
```

### 테스트 커버리지
- **Jacoco** 사용, `java-convention`에서 자동 설정
- **도메인 모듈 최소 커버리지: 80%** — 미달 시 빌드 실패
- **APP 모듈 최소 커버리지: 50%** — 미달 시 빌드 실패

### test-fixtures
- `java-test-fixtures` 플러그인으로 엔티티 팩토리 분리
- 매번 직접 생성하지 않고 fixture 팩토리를 통해 생성

```java
// testFixtures/java/com/moylog/domain/PostFixture.java
public class PostFixture {
    public static Post create() {
        return Post.create(1L, "제목", "내용");
    }
}
```

---

# 3. GitHub Convention

## 브랜치 전략

- **`main`이 근본 브랜치** (항상 배포 가능한 상태 유지)
- 이슈 먼저 생성 → `feature/<이슈번호>` 브랜치 생성 → PR → 최소 1 Approve → 머지
- `main` 직접 push 금지

---

## 커밋 컨벤션

커밋 메시지는 **`<type>: <subject>`** 형식을 사용하며, 아래 예시를 참고하여 상황에 맞는 타입을 선택해 주세요.

### 👉 커밋 메시지 예시

* **✨ feat: 새로운 기능 추가**
  * 사용자에게 새로운 가치를 주는 기능적 변화가 있을 때
  * *예시:* `feat: 로그인 시 이메일 중복 체크 기능 추가`
* **🛠️ fix: 버그 수정**
  * 코드 오류, 서버 에러, 잘못된 계산 로직 등을 해결했을 때
  * *예시:* `fix: 게시글 조회 시 댓글이 누락되는 오류 해결`
* **📝 docs: 문서 수정**
  * `README.md`, Swagger, 주석 등 문서만 바꿀 때
  * *예시:* `docs: API 환경 변수 설정법 업데이트`
* **🎨 style: 코드 포맷팅**
  * 로직 변경 없이 들여쓰기, 미사용 import 제거 등 코드 스타일만 다듬을 때
  * *예시:* `style: 불필요한 import 정리 및 코드 포맷팅 수정`
* **♻️ refactor: 코드 리팩토링**
  * 기능은 유지하되 가독성 향상, 책임 분리 등 구조를 개선할 때
  * *예시:* `refactor: 결제 로직 내 중복 코드를 공통 메서드로 분리`
* **✅ test: 테스트 관련**
  * 테스트 코드 신규 작성, 기존 테스트 보완 및 수정 시
  * *예시:* `test: 회원가입 성공/실패 케이스 단위 테스트 추가`
* **⚙️ chore: 설정 및 관리**
  * 의존성 추가(gradle), 패키지 구조 변경 등 로직 외 작업
  * *예시:* `chore: build.gradle 내 QueryDSL 의존성 추가`
* **👷 ci: 자동화 및 배포**
  * GitHub Actions, Docker 등 CI/CD 스크립트 수정 시
  * *예시:* `ci: GitHub Actions 빌드 실패 시 슬랙 알림 연동`

### 📂 패키지 구조 변경은 어떤 타입을 쓰나요?

| 상황        | 추천 Type        | 이유                                           |
|:----------|:---------------|:---------------------------------------------|
| **설계 개선** | **`refactor`** | 레이어 분리, 도메인별 패키징 등<br>코드의 구조적 설계를 개선하는 경우    |
| **단순 관리** | **`chore`**    | 패키지명 오타 수정, 빈 폴더 삭제 등<br>논리적 구조 변경이 없는 단순 정리 |

> **Tip:** 만약 패키지를 옮기면서 내부 로직(코드)도 함께 수정했다면, 고민 없이 **`refactor`** 를 사용한다.

---

### PR 컨벤션

> **PR 제목 형식 - \<type>: \<subject> (scope)**  
* 스코프는 이슈 번호로 표현한다.
  * 예시) feat: 기능추가 (#1)
- **최소 1명의 Approve** 필수
  - 팀원의 여유가 되지 않는다면 `@gemini review`로 리뷰 받는다.
- CI(빌드 & 테스트 & 커버리지 50%) 통과 필수
- **머지 방식: Squash and merge**

### 코드 리뷰 프로세스

1. PR 생성 시 CodeRabbit AI가 자동으로 전체 설계 리뷰 및 요약 생성
2. 팀원 리뷰가 늦어지거나 즉각적인 피드백이 필요할 경우 @gemini review 호출로 코드 상세 검수
3. 모든 자동화 도구(CI)와 AI 리뷰어의 피드백을 반영하여 최종 Squash and merge

---

# 4. 협업 및 자동화 도구

## 프로필 (application.yml)
- `application.yml` : 공통 설정
- `application-prod.yml` : 프로덕션 설정
- `application-test.yml` : 테스트 설정 (H2)

---

## 코드 품질 및 정적 분석

* JaCoCo: 테스트 커버리지 측정 및 빌드 실패 조건 설정 (Domain 80%, App 50%)
* Checkstyle: Google Java Style Guide 기반의 코드 컨벤션 자동 검사
* SonarQube: 코드 복잡도, 중복, 취약점 등을 대시보드로 통합 관리 및 기술 부채 추적

## 자동화 및 리뷰 어시스턴트

* Gradle Git Hook: 별도의 외부 의존성(Node.js) 없이 Gradle Task를 활용하여 Git Hook 관리
  * 메시지 작성 (prepare-commit-msg): 커밋 타입에 맞는 이모지를 자동으로 접두어에 추가.
  * 푸시 전 (pre-push): 전체 Test 수행 및 Jacoco 커버리지(Domain 80%, App 50%) 등 충족 여부 확인.
* CodeRabbit AI: PR 생성 시 아키텍처 관점의 전체적인 흐름 및 로직 설계 리뷰
  * 프로젝트의 컨텍스트를 이해하고, 모듈 간의 의존성이나 아키텍처 설계가 컨벤션에 부합하는지 '숲'을 보는 관점에서 리뷰 수행
* Gemini Assist: 리뷰어 부재 시 또는 개별 코드 단위의 즉각적인 피드백 담당
  * 메서드 가독성, 엣지 케이스 제안, Java 25 최신 문법 활용 제안 등 '나무'를 보는 관점에서 리뷰 수행