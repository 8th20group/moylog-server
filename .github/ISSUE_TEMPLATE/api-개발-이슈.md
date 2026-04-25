---
name: API 개발 이슈
about: API 개발 이슈 템플릿 예시입니다.
title: "[✨ FEATURE-API]"
labels: "✨ FEATURE, \U0001F534 PRIORITY:HIGH"
assignees: ''
type: Feature

---

<!-- 작업 우선순위 설정이 필요합니다. 기본값: 최상 -->
## 📌 개요

<!-- 도메인에 대한 이슈 번호를 남긴다. 예시) 게시글(#1) 생성 API --> 
* 어떤 API인지 설명 (ex: 회원 생성 API)

## 🔗 Endpoint

* Method:
* URL:

## 📥 Request

```json
{
}
```

## 📤 Response

### 성공

```json
{
  "data": null,
  "error": null,
  "timestamp": 생략
}
```

### 실패

```json
{
  "data": null,
  "error": {
    "code": null,
    "message": null,
    "detail": {
       [ field: null, reason: null ]  
     }
   },
  "timestamp": 생략
```

## 📊 상태 코드

* 200 OK:
* 201 Created:
* 400 Bad Request:
* 404 Not Found:

## 🔐 인증/인가

* 필요 여부 및 방식

## ✅ 완료 조건

* [ ] Controller 구현
* [ ] Service 연결
* [ ] Validation 적용
* [ ] 테스트 작성

## 💬 참고 사항

* 프론트 협업 내용 등
