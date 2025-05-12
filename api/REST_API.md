# REST API
- REST를 기반으로 만들어진 API
- REST (Representational State Transfer)는 웹에서 자원을 정의하고 자원에 대한 주소(URI)를 통해 접근하는 아키텍처 스타일
## REST
- HTTP URI를 통해 자원을 명시하고, 
- HTTP Method를 통해 
- HTTP Method를 사용해 해당 자원에 대한 CRUD 연산을 수행
### REST 구성요소
1. 자원 (Resource): HTTP URI
2. 자원에 대한 행위 : HTTP Method 

   | Method | 설명    | 예시 요청             |
   | ------ | ----- | ----------------- |
   | GET    | 조회    | `GET /users/1`    |
   | POST   | 생성    | `POST /users`     |
   | PUT    | 전체 수정 | `PUT /users/1`    |
   | PATCH  | 일부 수정 | `PATCH /users/1`  |
   | DELETE | 삭제    | `DELETE /users/1` |

3. 자원에 대한 행위의 내용 : HTTP Message Pay Load
- 클라이언트가 서버에 전달하는 데이터
- JSON 혹은 XML 형태
```
POST /users HTTP/1.1
Content-Type: application/json

{
"name": "Alice",
"email": "alice@example.com"
}
```

### REST 특징 (6가지)
- Server-Client (서버-클라이언트 구조)
  - 클라이언트와 서버는 역할이 명확히 분리되어야 함
  - 서버는 데이터 저장과 처리 기능을 담당함, 클라이언트에서는 사용자 인증이나 컨텍스트(세션, 로그인 정보)등을 직접 관리하는 구조
  - 역할 분리로 서로 간의 의존성이 줄어듦
- Stateless (무상태성)
  - 서버는 클라이언트의 상태를 저장하지 않음
  - 각 요청은 독립적으로 처리되어야 하고, 필요한 모든 정보는 요청 안에 포함되어야 함
  - 서비스의 자유도가 높아지고 서버에서 불필요한 정보를 관리하지 않음으로써 구현이 단순해짐
- Cacheable (캐시 처리 가능)
  - 웹 표준 HTTP 프로토콜을 그대로 사용하므로 웹에서 사용하는 기존의 인프라를 활용, 캐싱 기능을 적용할 수 있음
  - HTTP 프로토콜 표준에서 사용하는 Last-Modified 태그나 E-Tag를 이용하여 캐싱 구현이 가능
- Layered System(계층화)
  - 클라이언트는 REST API Server만 호출
  - REST Server는 다중 계층으로 구성될 수 있음
    - 로드밸런싱, 암호화 계층을 추가할 수 있음
  - 프록시, 게이트웨이 같은 네트워크 기반의 중간 매치를 사용할 수 있음
- Code-On-Demand(optional)
  - 클라이언트가 서버로부터 스크립트를 받아 실행할 수 있는 기능
- Uniform Interface(인터페이스 일관성)
  - URI로 지정한 Resource에 대한 조작을 통일되고 한정적인 인터페이스로 수행함
  - HTTP 표준 프로토콜에 따르는 모든 플랫폼에서 사용이 가능

## REST API 설계 규칙
1. URI는 명사, 복수형으로
```
GET /users       → 사용자 목록 조회
GET /users/42    → ID가 42인 사용자 조회
```
2. HTTP 메서드를 행위로 활용

   | 행위 | 메서드    | 예시            |
   |----|--------| ------------- |
   | 조회 | GET    | `GET /users/1` |
   | 생성 | POST   | `POST /users` |
   | 수정 | PUT    | `PUT /users/1` |
   | 삭제 | DELETE | `DELETE /users/1` |

3. 계층 관계는 URI로 표현
```
GET /users/42/orders      → 42번 사용자의 주문 목록 조회
GET /users/42/orders/7    → 42번 사용자의 7번 주문 상세 조회
```

### ✅ URI 설계 시 주의할 점

1. **슬래시(`/`) 구분자는 계층 관계를 나타내는 데 사용**
    - 예: `/users/1/orders/5` (1번 유저의 5번 주문)

2. **URI 마지막에 슬래시(`/`)를 포함하지 않는다**
    - REST에서는 `/users`와 `/users/`를 서로 다른 리소스로 인식할 수 있음
    - URI의 모든 글자는 고유한 리소스를 식별해야 함

3. **하이픈(`-`)은 단어 구분 시 가독성을 높이기 위해 사용**
    - 예: `/user-profile`, `/order-history`
    - URL에서 하이픈은 검색엔진에도 유리함

4. **밑줄(`_`)은 사용하지 않는다**
    - 대신 하이픈(`-`)을 사용
    - 밑줄은 가독성이 떨어지고 일부 브라우저/폰트에서는 잘 보이지 않을 수 있음

5. **URI는 소문자 사용을 권장**
    - 대소문자를 구분하므로, 대문자는 혼란을 유발할 수 있음
    - 일관된 규칙 유지가 중요

6. **파일 확장자는 URI에 포함하지 않는다**
    - `.json`, `.xml` 등의 확장자는 URI에 노출하지 않고, `Accept` 헤더로 포맷을 지정
    - 예: `Accept: application/json`

