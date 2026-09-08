# StompTest - Backend (Spring Boot STOMP Server)

Spring Boot 기반의 WebSocket & STOMP 프로토콜을 활용한 실시간 채팅 백엔드 연구 및 구현 프로젝트입니다.

---

## 1. 프로젝트 개요 (Project Overview)

본 프로젝트는 WebSocket과 STOMP(Simple Text Oriented Messaging Protocol) 기반의 메시징 서블릿 구조를 이해하고, 메모리 기반 메시지 브로커(SimpleBroker)를 통한 Pub/Sub 브로드캐스팅 라우팅 메커니즘을 백엔드에서 구현한 연구 프로젝트입니다.

- **주요 기능**: WebSocket / SockJS 엔드포인트 개설, STOMP 프로토콜 브로커 라우팅, 실시간 메시지 수신 및 구독 채널 브로드캐스팅
- **개발 환경 및 기술 스택**:
  - **Language**: Java 17
  - **Framework**: Spring Boot 3.3.2
  - **Modules**: `spring-boot-starter-web`, `spring-boot-starter-websocket`, `spring-messaging`
  - **Build Tool**: Gradle 8.x
  - **Utilities**: Lombok

---

## 2. 연구 목적의 자세한 내용 (Detailed Research Objectives)

### 2.1 HTTP와 WebSocket/STOMP 프로토콜 동작 차이 분석
- **기존 HTTP 프로토콜의 한계**: 클라이언트의 요청이 있을 때만 서버가 응답하는 비연결성(Stateless) 및 단방향 구조로 인해 실시간 커뮤니케이션 구현 시 폴링(Polling)으로 인한 서버 오버헤드가 발생함.
- **WebSocket 지속 연결(Persistent Connection)**: 최초 3-Way Handshake 및 HTTP Upgrade를 거친 후 양방향 전이중(Full-Duplex) TCP 통신 채널을 유지함으로써 커넥션 연결/해제 오버헤드를 최소화함.

### 2.2 STOMP (Simple Text Oriented Messaging Protocol) 도입 배경
- 순수 WebSocket은 바이너리/텍스트 프레임 수준의 낮은 레이어만 제공하므로, 상위 애플리케이션 수준의 메시지 구조(Header, Payload, Destination 지정) 및 라우팅을 직접 구현해야 함.
- STOMP 프로토콜을 도입하여 표준화된 프레임 구조(`CONNECT`, `SEND`, `SUBSCRIBE`, `MESSAGE` 등)를 사용하고, 메시지 브로커 기반의 발행/구독(Pub/Sub) 아키텍처를 용이하게 구축함.

### 2.3 SockJS Fallback 메커니즘 연구
- 구형 웹 브라우저나 WebSocket 프로토콜 접속을 차단하는 기업 방화벽/프록시 환경에서도 실시간 통신이 중단되지 않도록 HTTP Streaming, Long Polling 등으로 유연하게 전환(Fallback)되는 SockJS 호환 엔드포인트를 검증함.

### 2.4 Spring Messaging & Controller 라우팅 아키텍처 설계
- `@MessageMapping` 어노테이션 기반의 애플리케이션 목적지 프레픽스(`/app`)와 메시지 브로커의 구독 프레픽스(`/topic`, `/queue`) 간 분리 라우팅 메커니즘을 검증함.

---

## 3. 연구 결과의 자세한 내용 (Detailed Research Results)

### 3.1 WebSocket 및 STOMP 브로커 설정 (`StompConfig.java`)
- **엔드포인트 등록 (`registerStompEndpoints`)**:
  - `/chatting` 엔드포인트를 등록하여 클라이언트 핸드셰이크 요청을 수신함.
  - `.setAllowedOriginPatterns("*")`: CORS 제약을 해제하여 외부 프론트엔드 도메인과의 통신을 허용함.
  - `.withSockJS()`: WebSocket 미지원 환경을 위한 SockJS Fallback 지원을 활성화함.
- **메시지 브로커 설정 (`configureMessageBroker`)**:
  - `registry.enableSimpleBroker("/topic", "/queue")`: 내장 인메모리 SimpleBroker를 활성화하여 `/topic`(1:N 방송용) 및 `/queue`(1:1 전송용) 채널 구독 클라이언트에게 메시지를 전달함.
  - `registry.setApplicationDestinationPrefixes("/app")`: 클라이언트가 `/app`으로 시작하는 목적지로 전송한 메시지를 `@MessageMapping` 컨트롤러로 라우팅함.

### 3.2 메시지 수신 및 브로드캐스팅 컨트롤러 (`StompController.java`)
- **수신 라우팅 (`@MessageMapping("/chat/send")`)**:
  - 클라이언트가 `/app/chat/send` 라우트로 전송한 JSON Payload(`sender`, `contents`)를 수신함.
- **메시지 발행 (`SimpMessagingTemplate.convertAndSend`)**:
  - 수신된 메시지를 `/topic/1` 구독 채널로 즉시 전송하여 해당 토픽을 구독 중인 모든 클라이언트에게 실시간 브로드캐스팅을 성공적으로 수행함.

### 3.3 연구 결과 및 검증 요약
1. **커넥션 및 라우팅 성능 확보**: 인메모리 SimpleBroker를 통해 여러 세션이 동시 접속하더라도 대기 시간 없이 실시간 메시지 라우팅이 원활하게 동작함을 확인함.
2. **확장성 검증**: 단순 구조에서 향후 외부 전문 메시지 브로커(RabbitMQ, ActiveMQ, Redis Pub/Sub 등)로 전환하기 용이한 표준 STOMP 인터페이스 구조를 정립함.
