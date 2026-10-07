# Ontology E-commerce

온라인 리테일 도메인의 운영 데이터를 온톨로지와 에이전트 인터페이스로 모델링하는 개인 학습 프로젝트입니다.

상품, 재고, 주문, 배송을 단순한 CRUD 리소스로 다루는 대신, 도메인 객체와 관계·행위·감사 이력을 하나의 실행 가능한 모델로 구성하는 것을 목표로 합니다.

## Scope

현재 모델링 대상은 다음과 같습니다.

- `product`: 판매 상품과 가격
- `inventory`: 창고별 가용·예약 재고
- `order`: 주문 상태와 결제 상태
- `order_item`: 주문과 상품의 구성 관계
- `shipment`: 출고 및 배송 상태
- `customer`: 고객 식별자와 등급
- `proposal`: 에이전트가 생성한 변경 제안
- `audit_log`: 액션 실행 이력

첫 번째 운영 시나리오는 재고 부족으로 배송 지연 가능성이 있는 주문을 식별하고, 대체 창고 할당 또는 고객 안내를 제안하는 흐름입니다. 변경은 사람의 승인 경계를 통과한 경우에만 반영하도록 구성합니다.

## Architecture

```text
Cloud SQL for PostgreSQL
        ↓
Firebase SQL Connect
  schema / connectors / generated SDK
        ↓
TypeScript domain API
  ontology / actions / audit log
        ↓
OpenAI Agents SDK
  domain tools / proposals / human approval
```

Firebase SQL Connect는 Cloud SQL for PostgreSQL 위에 애플리케이션용 GraphQL 스키마와 커넥터를 제공하는 접근 계층으로 사용합니다. 실제 도메인 규칙과 변경 권한은 TypeScript API와 액션 레이어에서 관리합니다.

## Technology

- TypeScript
- Node.js 22+
- Hono
- Firebase SQL Connect
- Cloud SQL for PostgreSQL
- OpenAI Agents SDK (`@openai/agents`)
- Vite / React (운영 화면)

## Repository layout

```text
apps/
  api/              # 온톨로지 API와 도메인 액션
  agents/           # 에이전트와 도메인 도구
  web/              # 운영 화면
dataconnect/
  schema/           # SQL Connect schema
  connector/        # query / mutation connector
db/
  seeds/            # 재현 가능한 초기 데이터
docs/
  decisions/        # 설계 결정과 변경 기록
```

## Design notes

- 도메인 객체는 데이터베이스 테이블과 1:1로 고정하지 않고, 메타데이터를 통해 조회 가능한 온톨로지로 노출합니다.
- 읽기 도구와 쓰기 액션을 분리합니다.
- 에이전트가 직접 상태를 변경하지 않고, 제안(`proposal`)과 승인 단계를 거칩니다.
- 모든 액션은 호출자, 입력, 결과, 승인 근거를 감사 로그에 남깁니다.
- SQL Connect는 데이터 접근과 타입 생성을 담당하고, 도메인 정책은 애플리케이션 레이어에 둡니다.

## Status

현재는 도메인 범위와 시스템 경계를 정리한 초기 단계입니다. 다음 작업은 SQL Connect schema와 상품·재고·주문 기본 모델을 추가하는 것입니다.
