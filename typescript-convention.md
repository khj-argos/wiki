# TypeScript Convention

## 범위

이 문서는 TypeScript의 타입 표기와 값 선언 방식을 정합니다. 함수 길이, 분기, 상속, 네이밍과 주석 같은 언어 공통 규칙은 다루지 않습니다.

## `any`

`any`를 사용하지 않습니다. 타입을 알면 구체적인 타입을 사용하고, 알 수 없는 외부 입력은 `unknown`으로 받은 뒤 검증하거나 좁혀서 사용합니다.

## `type`과 `interface`

타입 선언에는 기본적으로 `type`을 사용합니다. 클래스가 `implements`할 계약도 `type`으로 선언할 수 있으므로 그 이유만으로 `interface`를 만들지 않습니다.

`interface`는 외부 라이브러리 타입을 선언 병합으로 확장해야 할 때만 사용합니다.

## 닫힌 값 집합

TypeScript `enum`을 사용하지 않습니다. 상태, 역할, 종류처럼 닫힌 값 집합은 `as const` 객체로 선언하고 `ValueOf<typeof CONSTANT>`로 값 union 타입을 파생합니다. `ValueOf<T>`는 `T[keyof T]`를 나타내는 공통 유틸리티 타입으로 각 저장소의 공통 타입 모듈에 둡니다.

닫힌 값 집합은 해당 도메인의 상수 파일 하나만 source of truth로 사용합니다. 제품 코드와 테스트에서 같은 값 리터럴을 다시 쓰지 않고 그 상수를 참조합니다.

## 타입 파생

같은 bounded context에서 의미와 lifecycle이 같은 필드는 canonical owner type 하나가 소유합니다. 다른 계층이나 기능에서 필요한 부분 형태를 새로 선언하지 않고 owner type에서 파생합니다.

| 상황 | 작성 방식 |
| --- | --- |
| 일부 필드만 선택 | `Pick<T, K>` |
| 일부 필드 제외 | `Omit<T, K>` |
| 모든 필드를 선택적으로 변경 | `Partial<T>` |
| 모든 필드를 필수로 변경 | `Required<T>` |
| 모든 필드를 읽기 전용으로 변경 | `Readonly<T>` |
| 키와 값의 매핑 | `Record<K, T>` |
| canonical field에 새 필드 추가 | 기존 키와 겹치지 않는 필드만 intersection으로 확장 |
| 더 좁은 상태나 불변조건 표현 | canonical owner를 기반으로 refinement type을 만들고 owner와 invariant를 명시 |
| 기존 필드의 의도적인 교체 | `Omit<Base, keyof Replacement> & Replacement`를 사용하고 교체 이유와 owner 변경 여부를 명시 |

canonical field를 근거 없이 `string`, `boolean` 같은 넓은 타입으로 다시 선언하지 않습니다. 교차 타입을 기존 키의 재선언 수단으로 사용하지 않습니다. 값 집합이나 필드의 owner가 없다면 소비 타입을 복제하기 전에 canonical owner를 먼저 정합니다.

## Type assertion

제품 코드에서 타입을 강제로 바꾸는 `as` assertion을 직접 사용하지 않습니다. 닫힌 값 집합을 선언하는 `as const`는 이 금지에 포함하지 않습니다.

그 밖의 assertion이 불가피하면 타입 helper 유틸리티 내부로 한정하고, 해당 assertion이 필요한 이유를 주석으로 남깁니다.
