# StompTest - Frontend (HTML5 / SockJS / STOMP.js Web Client)

SockJS 및 STOMP.js 라이브러리를 활용한 웹 브라우저 기반 실시간 채팅 클라이언트 연구 및 구현 프로젝트입니다.

---

## 1. 프로젝트 개요 (Project Overview)

본 프로젝트는 웹 브라우저 환경에서 SockJS 프로토콜 기반으로 백엔드 STOMP 서버와 WebSocket/SockJS 연결을 맺고, STOMP 프로토콜 프레임을 송수신하여 실시간 채팅 및 비동기 DOM 렌더링을 수행하는 프론트엔드 연구 프로젝트입니다.

- **주요 기능**: 사용자 닉네임 설정, STOMP Handshake & 커넥션 유지, 토픽(`/topic/1`) 구독(Subscribe), 메시지 발행(Publish `/app/chat/send`), 발신자/수신자 메시지 동적 CSS 스타일링 렌더링
- **개발 환경 및 기술 스택**:
  - **Language**: HTML5, CSS3, JavaScript (ES5/ES6)
  - **Libraries**:
    - jQuery 3.6.0 (DOM 조작 및 이벤트 바인딩)
    - SockJS Client 1.x (WebSocket 대체 통신 레이어)
    - Stomp.js 2.3.3 (STOMP 프로토콜 클라이언트 래퍼)

---

## 2. 연구 목적의 자세한 내용 (Detailed Research Objectives)

### 2.1 클라이언트 단의 STOMP 통신 생태계 분석
- 웹 브라우저 환경에서 백엔드 STOMP 엔드포인트(`/chatting`)에 SockJS 커넥션을 생성하고, STOMP 프로토콜 레이어(`Stomp.over(socket)`)로 래핑하여 표준 프레임을 처리하는 메커니즘 검증.

### 2.2 실시간 비동기 메시지 수신 및 DOM 갱신 반응성 연구
- 서버 채널 구독(`stompClient.subscribe('/topic/1', ...)`)을 통해 비동기 이벤트 콜백 함수가 호출될 때의 UI 응답 속도 및 메시지 파싱 정합성 검증.
- 웹 페이지 전체 리로드(Full Page Reload) 없이 채팅 내역이 동적으로 누적 렌더링되는 Single-Page 실시간 UI 반응성 측정.

### 2.3 클라이언트 메시지 식별 및 시각적 UX 분리 설계
- 발신된 메시지 Payload(`sender`, `contents`)를 파싱하여 메시지 생성 주체가 사용자 본인(`userId`)인지 다른 사용자인지에 따라 CSS 클래스(`.me`, `.other`)를 분리 적용하여 사용자 경험(UX) 극대화.

---

## 3. 연구 결과의 자세한 내용 (Detailed Research Results)

### 3.1 통신 연결 및 핸드셰이크 구현 (`ChatTest.html`)
- **사용자 식별**: `window.prompt("사용자 닉네임?")`을 통한 대화형 사용자 닉네임 수집.
- **SockJS 커넥션 생성**: `var socket = new SockJS("/chatting");`을 사용하여 Spring 백엔드의 SockJS 엔드포인트와 커넥션 형성.
- **STOMP 프로토콜 바인딩**: `stompClient = Stomp.over(socket);`로 STOMP 프로토콜 핸들러를 래핑한 후 `stompClient.connect({}, successCallback, errorCallback)`으로 핸드셰이크 완료.

### 3.2 메시지 구독 및 동적 DOM 렌더링
- **토픽 구독 (`subscribe`)**:
  - `stompClient.subscribe('/topic/1', function (e) { showMessage(JSON.parse(e.body)); });`
  - `/topic/1` 구독을 통해 브로드캐스트되는 JSON 형태의 데이터(`sender`, `contents`)를 수신하고 `showMessage` 함수 호출.
- **UI 렌더링 (`showMessage`)**:
  - `data.sender === userId` 조건을 평가하여 본인 메시지는 파란색(`.me`), 타인 메시지는 빨간색(`.other`) 텍스트로 채팅 박스(`#chatting`) 하단에 추가(`append`).

### 3.3 메시지 전송 및 키 이벤트 처리
- **메시지 전송 (`send`)**:
  - 입력 폼(`#msg`)에 작성된 텍스트와 닉네임을 파싱하여 JSON Payload 구성 후 `stompClient.send("/app/chat/send", {}, JSON.stringify(data));`를 통해 백엔드 라우터로 전송.
  - 전송 완료 후 입력창 값 자동 초기화.
- **사용자 편의 기능**: Enter 키 입력 시 메시지가 즉시 발송되도록 keypress 이벤트 바인딩.

### 3.4 연구 결과 및 검증 요약
1. **양방향 실시간 메시징 검증**: 백엔드 브로드캐스팅과 결합하여 다중 브라우저 탭 간 실시간 1:N 메시지 전달 및 즉각적인 UI 반향 확인.
2. **SockJS Fallback 프론트엔드 호환성 확인**: 브라우저 통신 제약 환경에서도 SockJS 래퍼가 안정적인 세션을 유지함을 확인함.
