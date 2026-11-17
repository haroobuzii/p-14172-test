# p-14172-test

스프링부트로 만든 **게시글/댓글 REST API** 과제입니다.
[강의 페이지](https://www.slog.gg/p/14172/raw)를 참고해서, 기능마다 커밋을 나눠 구현했습니다.

## 과제 항목

- [x] 프로젝트 새로 생성
- [x] 글 3건, 댓글 5건 자동생성
- [x] 글 작성 기능/테스트케이스
- [x] 글 삭제 기능/테스트케이스
- [x] 댓글 작성 기능/테스트케이스
- [x] 댓글 삭제 기능/테스트케이스
- [x] 리포지터리 제출

## 기술 스택

| 구분 | 사용 기술 |
|---|---|
| 언어 | Java 25 |
| 프레임워크 | Spring Boot 4.1.2 (Web MVC, Data JPA, Validation) |
| DB | H2 (개발: 파일 DB, 테스트: 인메모리 DB) |
| 기타 | Lombok, Gradle |
| 테스트 | JUnit 5, MockMvc |

## 실행 방법

```bash
./gradlew bootRun
```

- 서버 주소: `http://localhost:8080`
- 처음 실행하면 샘플 데이터(글 3건, 댓글 5건)가 자동으로 생성됩니다.
- H2 콘솔: `http://localhost:8080/h2-console`
  - JDBC URL: `jdbc:h2:./db_dev;MODE=MySQL`
  - User Name: `sa` / Password: 없음

샘플 데이터를 처음 상태로 되돌리려면 서버를 끄고 `db_dev.*` 파일을 지운 뒤 다시 실행하세요.

## 테스트 실행

```bash
./gradlew test
```

- `test` 프로필(인메모리 DB)로 실행되어 개발용 DB에는 영향을 주지 않습니다.
- 각 테스트는 `@Transactional`로 끝나면 롤백되므로 서로 영향을 주지 않습니다.

## API

### 글

| 기능 | 메서드 | 주소 | 성공 응답 |
|---|---|---|---|
| 글 작성 | `POST` | `/api/v1/posts` | `201` |
| 글 삭제 | `DELETE` | `/api/v1/posts/{id}` | `200` |

### 댓글

| 기능 | 메서드 | 주소 | 성공 응답 |
|---|---|---|---|
| 댓글 작성 | `POST` | `/api/v1/posts/{postId}/comments` | `201` |
| 댓글 삭제 | `DELETE` | `/api/v1/posts/{postId}/comments/{id}` | `200` |

### 요청 예시

글 작성

```json
{
  "title": "제목",
  "content": "내용"
}
```

댓글 작성

```json
{
  "content": "내용"
}
```

### 응답 형식 (RsData)

조회를 제외한 모든 응답은 `resultCode`, `msg`, `data`를 담은 공통 형식으로 응답합니다.
`resultCode`의 앞자리(`201-1`의 `201`)가 HTTP 응답코드로 사용됩니다. (`ResponseAspect`)

글 작성 성공

```json
{
  "resultCode": "201-1",
  "msg": "4번 글이 작성되었습니다.",
  "data": {
    "id": 4,
    "createDate": "2026-11-17T10:00:00.000000",
    "modifyDate": "2026-11-17T10:00:00.000000",
    "title": "제목",
    "content": "내용"
  }
}
```

글 삭제 성공 (`data` 없음)

```json
{
  "resultCode": "200-1",
  "msg": "1번 글이 삭제되었습니다.",
  "data": null
}
```

### 실패 응답

| 상황 | HTTP | resultCode | msg |
|---|---|---|---|
| 입력값 검증 실패 (예: 제목 없음) | `400` | `400-1` | `title-NotBlank-must not be blank` 처럼 `필드-규칙-메시지` 형태 |
| JSON 형식이 잘못됨 | `400` | `400-1` | `요청 본문이 올바르지 않습니다.` |
| 없는 글/댓글 | `404` | `404-1` | `해당 데이터가 존재하지 않습니다.` |

입력값 규칙

- 글 제목: 2 ~ 100자, 글 내용: 2 ~ 5000자
- 댓글 내용: 2 ~ 100자

## 프로젝트 구조

```
src/main/java/com/back
├── BackApplication.java                 # 앱 시작점
├── domain/post
│   ├── post
│   │   ├── controller/ApiV1PostController.java        # 글 API
│   │   ├── dto/PostDto.java                           # 글 응답용 데이터
│   │   ├── entity/Post.java                           # 글 엔티티
│   │   ├── repository/PostRepository.java
│   │   └── service/PostService.java
│   └── postComment
│       ├── controller/ApiV1PostCommentController.java # 댓글 API
│       ├── dto/PostCommentDto.java                    # 댓글 응답용 데이터
│       └── entity/PostComment.java                    # 댓글 엔티티
└── global
    ├── aspect/ResponseAspect.java                     # resultCode → HTTP 응답코드
    ├── globalExceptionHandler/GlobalExceptionHandler.java  # 400, 404 처리
    ├── initData/BaseInitData.java                     # 샘플 데이터 생성
    ├── jpa/entity/BaseEntity.java                     # id, 작성일, 수정일 공통
    └── rsData/RsData.java                             # 공통 응답 형식
```

## 커밋 내역

| 순서 | 커밋 | 내용 |
|---|---|---|
| 1 | 프로젝트 생성 | Spring Boot, JPA, H2, validation 설정. dev/test 프로필 분리 |
| 2 | 글 3건, 댓글 5건 자동생성 | `BaseEntity`, `Post`, `PostComment` 엔티티, DTO, 샘플 데이터 생성과 개수 테스트 |
| 3 | 글 작성 기능/테스트케이스 | `POST /api/v1/posts`, `RsData`, `ResponseAspect`(201 응답), 예외 처리 |
| 4 | 글 삭제 기능/테스트케이스 | `DELETE /api/v1/posts/{id}`, 글 삭제 시 댓글도 함께 삭제(`CascadeType.REMOVE`) |
| 5 | 댓글 작성 기능/테스트케이스 | `POST /api/v1/posts/{postId}/comments`, 새 댓글 id를 위해 `flush` 후 응답 |
| 6 | 댓글 삭제 기능/테스트케이스 | `DELETE /api/v1/posts/{postId}/comments/{id}`, `orphanRemoval`로 삭제 |

## 테스트 목록 (총 12개)

| 테스트 클래스 | 테스트 |
|---|---|
| `BackApplicationTests` | 앱이 정상적으로 켜지는지 |
| `BaseInitDataTest` | 샘플 데이터가 글 3건, 댓글 5건인지 |
| `ApiV1PostControllerTest` | 글 작성 성공(201) / 제목 없이 작성(400) / 잘못된 JSON(400) / 글 삭제 성공 / 없는 글 삭제(404) |
| `ApiV1PostCommentControllerTest` | 댓글 작성 성공(201) / 내용 없이 작성(400) / 없는 글에 작성(404) / 댓글 삭제 성공 / 없는 댓글 삭제(404) |
