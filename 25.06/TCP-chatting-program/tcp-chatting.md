# TCP 채팅 프로그램 구현 + DOCKER 컨테이너화
<br>
<br>

## 📌 왜 TCP로 구현할까?
TCP는 **신뢰성 있는 연결**을 보장해주는 프로토콜이다. <br>
채팅처럼 **메시지가 순서대로, 왜곡없이 정확하게 전달되어야 하는** 경우 TCP가 적합하다. <br>

<br>
<br>


![서버와 클라이언트](https://github.com/solji622/LevelUp-Study/blob/7dca52f34bd1a609a281f7a7e329a1793e426d27/25.06/TCP-chatting-program/asset/server_client.png)
<br>
기본적으로 서버와 클라이언트는 1:1연결을 전제로 한다. <br>
하지만 구현하려는 채팅 프로그램은 나와 상대방, 최소 두 클라이언트가 하나의 서버에 연결되어야 한다. <br>
이런 경우, 여러 클라이언트를 서버는 어떻게 처리할까? <br>

<br>
<br>

## 📌 멀티 스레드(Multi-threading)
각각의 클라이언트 연결을 처리하기 위해 새로운 스레드를 생성하는 것 <br>
여러 클라이언트와 1:1로 각각의 연결을 따로 맺고, 그 연결들을 멀티 스레드로 관리하는 것이다. <br>
<br>

> **스레드(Thread)란?** <br>
> 프로세스 내에서 작업을 수행하는 단위로, 독립적인 실행 흐름을 가진다. <br>
<br>
<br>

<br>

## 📌 로컬 TCP 채팅 프로그램 구현
### 1. Server
```python
import socket
import threading

# 연결된 클라이언트 소캣 저장 리스트
clients = []

# 한 클라이언트가 보낸 메시지를, 나머지 클라이언트들에게 뿌려주는 함수
def broadcast(message, sender_soc):
    for client in clients:

        # 이미 메시지가 전송된 클라이언트 제외 모든 클라이언트들에게 메시지 전송
        if client != sender_soc:
            try:
                client.send(message)
            except:
                client.close()
                clients.remove(client)
```
```python 
# 한 클라이언트가 서버와 통신하는 모든 과정을 처리하는 함수 (1:1 통신 담당)
def handle_client(client_soc):
    while True:
        try:
            message = client_soc.recv(1024)
            if not message:
                break
            # 메시지 받으면 broadcast()로 뿌림림
            broadcast(message, client_soc)
        except:
            break
    # 오류나 연결 종료 시 리스트에서 제거
    clients.remove(client_soc)
    client_soc.close()
```
``` python
# 서버 소캣 생성 > 바인딩 > 연결 대기
server_soc = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_soc.bind(('0.0.0.0', 55555))
server_soc.listen()

print('Server: port 55555로 서버 실행 중...')

while True: 
    client_soc, addr = server_soc.accept()
    print(f'접속: {addr}')
    # 새로운 클라이언트 접속 시 clients에 추가
    clients.append(client_soc)

    # 새로운 스레드를 만들어서 클라이언트를 처리하는 코드
    thread = threading.Thread(target=handle_client, args=(client_soc,))
    thread.start()
```
<br>
<br>


### 2. client
```python
import socket
import threading

# 서버로부터 메시지를 받는 스레드
def receive_messages(sock):
    while True:
        try: 
            message = sock.recv(1024).decode()
            if not message:
                break
            print(message)
        except:
            break
```
```python
name = input('닉네임 입력: ')
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('chat_server', 55555))

# 백그라운드에서 서버로부터 메시지 계속 듣는 스레드 실행
threading.Thread(target=receive_messages, args=(client,), daemon=True).start()
```
```python
# 사용자 입력을 서버로 전송, exit 입력 시 종료
while True:
    try:
        msg = input()
        if msg.lower() == 'exit':
            break
        full_msg = f'{name}: {msg}'
        client.sendall(full_msg.encode())
    except:
        break
client.close()
```
<br>
<br>
<br>
<br>

## 📌 Docker 컨테이너화

### 💡 왜 Docker로 컨테이너화를 할까?
Docker를 사용하면 서버와 클라이언트를 별도의 컨테이너에서 **독립적으로 실행**할 수 있다. <br>
이건 실제 서비스처럼 각 클라이언트가 서로 다른 컴퓨터에서 접속하는 상황을 **내 컴퓨터 안에서 간단하게 시뮬레이션** 할 수 있게 도와준다. <br>

또한, Python & 패키지 설치가 필요 없고 코드를 복사하여 바로 실행한다는 점에서 환경 구성도 단순해진다.

<br>


컨테이너화를 위해서는 Dockerfile이 필요하다 <br>
> **Dockerfile이란?** Docker 이미지를 만들 때 활용하는 것, 즉 이미지를 만들게 해주는 파일이다.
<br>

### 1. server
```Dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY server.py .

EXPOSE 55555

# 컨테이너 실행 시 `python server.py` 실행
CMD ["python", "server.py"]
```
<br>

### 2. client
```Dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY client.py .

CMD ["python", "client.py"]
```
<br>

### 3. docker-compose
```yml
services:
  chat_server:
    build: ./server
    container_name: chat_server
    ports:
      - 55555:55555
    networks:
      - chatnet

  chat_client:
    build: ./client
    # 입력 함수와 가상 터미널 사용 가능으로 지정
    stdin_open: true
    tty: true
    networks:
      - chatnet

networks:
  chatnet:
```

<br>
<br>
<br>
<br>

## 📌 프로그램 실행
```bash
docker compose build

# 서버 컨테이너 실행
docker compose up chat_server

# 클라이언트 컨테이너 실행
docker compose run --rm chat_client
```
클라이언트의 경우 여러 개의 터미널을 열어 실행할 수 있다. <br>
서로 다른 닉네임을 입력하여 메시지를 보내보면 실시간으로 주고받기가 가능한 걸 볼 수 있다. <br>

<br>
<Br>
<br>
<br>

***

<br>
<Br>
<br>
<br>


