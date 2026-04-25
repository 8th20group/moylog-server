# Moylog Backend Style Guide

## 🏗️ 1. Pure Domain Model
- **Rich Domain Model**: 데이터를 가진 모델을 지양하고, 비즈니스 로직은 최대한 모델 내부에서 수행한다.
- **Ownership**: 기획서에 명시되지 않은 엣지 케이스(예외 상황)를 도메인 레벨에서 먼저 고려했는지 체크한다.

## ☕ 2. Java & Spring Boot Convention
- **No `var`**: 타입 추론(`var`)을 사용하지 않고 명시적인 타입을 선언한다.
- **DTO Naming**: `dto` 패키지 내 클래스는 반드시 `Request`, `Response` 접미사를 붙인다.
- **Lombok**: `@Data` 사용은 금지하며, 불변성을 위해 `@Getter`와 생성자 주입을 권장한다.
- **Final Field**: 변경되지 않는 모든 필드와 파라미터에는 `final`을 적극적으로 사용한다.

## 🧪 3. 테스트 코드 (Testing)
- **English Method Name**: 테스트 메서드 명은 반드시 **영어**로 작성하며 CamelCase를 따른다.
- **No Line Limit**: 일반 코드는 짧게 유지하되, 테스트 코드는 가독성을 위해 120줄을 초과해도 허용한다.
- **Independent Test**: 각 테스트는 독립적이어야 하며, 실제 DB 대신 가급적 가짜 객체(Mock)나 인메모리 환경을 고려한다.

## 🛠️ 4. 에러 핸들링 및 기타
- **Custom Exception**: 표준 예외보다 비즈니스 의미가 담긴 커스텀 예외 클래스를 활용한다.
- **Optional**: `Optional.get()`을 직접 호출하지 않고, `orElseThrow` 등을 통해 안전하게 처리한다.
- **Checkstyle**: 코드 스타일은 사전에 설정된 Checkstyle 규칙을 준수하며, AI는 로직의 의도에 집중한다.