
## Socket이란?

소켓은 네트워크 상에서 돌아가는 두 개의 프로그램 간 양방향 통신을 도와주는 통로 역할을 하는 TCP/IP 기반의 통신 방식입니다.
**→ 양방향 통신을 도와주는 엔드 포인트(EndPoint)**

> EndPoint: IP address + port   
> ▶ 모든 TCP 연결은 2개의 엔드 포인트로만 식별 된다 

### 소켓의 주요 프로토콜

1. **TCP (Transmission Control Protocol)**
   - 연결 지향적 프로토콜
   - 데이터 전송의 신뢰성 보장
   - 데이터의 순서 보장
   - 속도는 상대적으로 느림
   - 예: 웹 브라우징, 이메일, 파일 전송

2. **UDP (User Datagram Protocol)**
   - 비연결 지향적 프로토콜
   - 데이터 전송의 신뢰성 보장하지 않음
   - 데이터의 순서 보장하지 않음
   - 속도가 빠름
   - 예: 실시간 스트리밍, 온라인 게임

### 소켓의 종류

**Server Socket(서버 소켓)**
: 서버 프로그램에서 사용하는 소켓으로 클라이언트로부터 요청을 기다리다가 요청이 오면 클라이언트로 요청을 맺는다.  
  클라이언트 소켓 간 연결을 도와준다.

**Client Socket(클라이언트 소켓)**
: 클라이언트에서 사용하는 소켓으로 생성하고 서버 프로그램으로 연결 요청을 하고 필요에 따라 데이터를 전송한다.

## 소켓 통신

소켓을 통해 Server와 Client가 특정 Port를 통해 실시간으로 양방향 통신 하는 방식

  → Streaming 중계나 실시간 채팅, 게임 등과 같이 즉각적으로 정보를 주고받는 경우에 사용

### 소켓 통신 방법

1. Server에서는 Port를 열고 Client의 접속을 기다린다.
2. Client는 Server의 IP와 Port에 접속한다. → Server와 Client 연결
3. Server와 Client 간의 통신은 Send, Receive의 형태로 주고받는다.
4. 그리고 통신이 끝나면 close()로 접속을 끊는다.

<br> 

## Web Socket이란?

WebSocket은 서버와 클라이언트 간에 Socket Connection을 유지해서 언제든 양방향 통신 또는 데이터 전송이 가능하도록 하는 기술입니다.

### HTTP vs WebSocket

1. **연결 방식**
   - HTTP: 요청-응답 후 연결 종료 (Stateless)
   - WebSocket: 연결을 계속 유지 (Stateful)

2. **통신 과정**
   - HTTP: 매 요청마다 새로운 연결 수립
   - WebSocket: 
     - 최초 접속 시에만 HTTP 사용 (HandShake)
     - 이후 WebSocket 프로토콜로 전환
     - 한번 연결된 후 계속 통신 가능

### WebSocket의 특징

- HTML 웹 표준 기술
- 매우 빠르게 작동하며 통신할 때 아주 적은 데이터를 이용
	- 프레임 헤더 크기가 2~14 bytes 정도로 매우 작음
	- 실제 데이터(payload)만 주고받음
- 이벤트를 단순히 듣고 보내는 것만 가능

### WebSocket의 한계점

1. **브라우저 호환성**
   - 구버전 브라우저에서 지원하지 않을 수 있음
   - 모바일 브라우저에서의 제한적 지원

2. **서버 부하**
   - 다수의 영구적 연결로 인한 서버 리소스 부담
   - 적절한 부하 관리 필요

3. **연결 관리**
   - 네트워크 문제로 인한 연결 끊김
   - 재연결 로직 직접 구현 필요
   - 상태 관리의 복잡성

<br>

## Socket.io 란?

WebSocket을 기반으로 자바스크립트를 이용하여 브라우저 종류에 상관없이 실시간 웹을 구현할 수 있도록 도와주는 라이브러리

### Socket.io의 주요 특징

1. **자동 재접속**
   - 연결이 끊어졌을 때 자동으로 재연결 시도
   - 커스텀 재연결 로직 구성 가능

2. **Fallback 지원**
   - WebSocket을 지원하지 않는 환경에서 다른 방식으로 대체
   - Long Polling, Flash Socket 등으로 자동 전환

3. **멀티플렉싱**
   - 하나의 연결로 여러 채널 통신 가능
   - 효율적인 리소스 사용

4. **Room과 Namespace**
   - 방 개념을 이용한 그룹 통신
   - Namespace를 통한 논리적 채널 분리

### Room 기능

- 서버에서 연결된 소켓(사용자)들을 세밀하게 관리할 수 있음
- Broadcasting 기능으로 특정 그룹에만 메시지 전송 가능
- Server만 Room을 관리할 수 있으며, Client는 참가/탈퇴만 가능

![Socket.IO Room 구조](https://blog.kakaocdn.net/dn/GeLJd/btrJEMkBknL/brFCLRytEFcpi2uHE5LRkk/img.png)


### 주요 메서드

**1. 연결 관련**

```javascript
io.on('connection', function(socket) {  
  // 신규 클라이언트 접속
}); 
```

**2. 메시지 수신**

```javascript
socket.on('event_name', function(data) { ... });
// event_name: 클라이언트가 메시지 송신 시 지정한 이벤트 명
// function: 이벤트 핸들러. 핸들러 함수의 인자로 클라이언트가 송신한 메시지 전달
```

**3. 메시지 송신**

| **명령어** | **설명** |
|------------|----------|
| io.emit(event_name, msg) | 접속된 모든 클라이언트에게 메시지 전송 |
| socket.emit(event_name, msg) | 메시지를 전송한 클라이언트에게만 메시지 전송 |
| socket.broadcast.emit(event_name, msg) | 메시지를 전송한 클라이언트를 제외한 모든 클라이언트에게 메시지 전송 |
| io.to(id).emit(event_name, msg) | 특정 클라이언트에게만 메시지 전송 (id는 socket 객체의 id 속성 값) |

**4. Room 관련**

| **명령어** | **설명** |
|------------|----------|
| socket.join(room) | 특정 room에 socket 연결 |
| socket.leave(room) | 특정 room에서 socket 연결 해제 |
| io.to(room).emit(event_name, msg) | 특정 room의 모든 클라이언트에게 메시지 전송 |
| socket.broadcast.to(room).emit() | 특정 room의 다른 클라이언트들에게 메시지 전송 |
