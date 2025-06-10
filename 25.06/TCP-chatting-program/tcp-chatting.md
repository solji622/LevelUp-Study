# TCP 채팅 프로그램 구현 + DOCKER 컨테이너화
<br>
<br>


![서버와 클라이언트](https://github.com/solji622/LevelUp-Study/blob/7dca52f34bd1a609a281f7a7e329a1793e426d27/25.06/TCP-chatting-program/asset/server_client.png)
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

clients = []

def broadcast(message, sender_soc):
    for client in clients:
        if client != sender_soc:
            try:
                client.send(message)
            except:
                client.close()
                clients.remove(client)

def handle_client(client_soc):
    while True:
        try:
            message = client_soc.recv(1024)
            if not message:
                break
            broadcast(message, client_soc)
        except:
            break
    clients.remove(client_soc)
    client_soc.close()

server_soc = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

server_soc.bind(('0.0.0.0', 55555))
server_soc.listen()

print('Server: port 55555로 서버 실행 중...')

while True: 
    client_soc, addr = server_soc.accept()
    print(f'접속: {addr}')
    clients.append(client_soc)
    thread = threading.Thread(target=handle_client, args=(client_soc,))
    thread.start()
```
<br>
<br>

### 2. client
```python
import socket
import threading

def receive_messages(sock):
    while True:
        try: 
            message = sock.recv(1024).decode()
            if not message:
                break
            print(message)
        except:
            break

name = input('닉네임 입력: ')
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('localhost', 55555))

threading.Thread(target=receive_messages, args=(client,), daemon=True).start()

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

## 📌 컨테이너화
컨테이너화를 위해서는 Dockerfile이 필요하다 <br>
> **Dockerfile이란?** Docker 이미지를 만들 때 활용하는 것, 즉 이미지를 만들게 해주는 파일이다.
<br>

### 1. server
```Dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY server.py .

EXPOSE 55555

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
    stdin_open: true
    tty: true
    networks:
      - chatnet

networks:
  chatnet:
```
