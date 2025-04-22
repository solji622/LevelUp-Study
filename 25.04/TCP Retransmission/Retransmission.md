# TCP Retransmission (TCP 재전송)

<br>
<Br>

## 📌 TCP 재전송은 왜 일어나는가?
TCP는 **신뢰성 있는 통신**이 가능해야하기에 자신이 보낸 데이터에 대해서 <br>
상대방이 받았다는 의미의 응답 패킷을 다시 받아야 통신이 정상적으로 이루어졌다고 생각한다. <br>

때문에 만약 자신이 보낸 데이터에 대한 응답 패킷을 받지 못하면 **패킷이 유실되었다** 판단하고 <Br>
보냈던 패킷을 다시 한번 보내게 되는데, 이 과정을 **TCP 재전송**이라 한다.

<img src="https://github.com/solji622/LevelUp-Study/blob/0571fcd58957007b2e2657bc220273863b40f908/25.04/TCP%20Retransmission/asset/retransmission-program.webp" width="70%">


<br>
<br>
<br>

## 📌 재전송이 발생하는 원인들
#### 1. 패킷 유실
> 송신자는 패킷을 보냈지만, 중간에 유실되어 수신 측에 도달하지 못한 경우 <br>
> 수신 측은 패킷이 도착하지 않아 ACK를 보낼 수 없고 송신자의 timer는 만료된다. 

#### 2. ACK 유실
> 송신자가 패킷을 보냈고 수신 측에서 응답으로 ACK를 보냈으나 중간에 유실된 경우 <br>
> 송신자에게 ACK가 도착하지 못하고 timer도 만료된다. <br>

#### 3. Early Timeout
> ACK 유실과 비슷하나 송신자의 timer가 만료된 후에 ACK가 도착한 경우 <br>

<br>

재전송이 발생하는 상황들에게서 공통점이 하나 있는데, **timeout 시간 안에 ACK를 받지 못했다는 것이다.** <br>
이 경우에 TCP가 할 수 있는 최선의 방법은 Timeout 시간을 정의하는 것인데 <br>
정의할 때 기준이 되는 시간에는 **RTO**라는 개념을 사용한다 <br>

<br>
<br>
<br>
<br>


## 📌 RTO(Retransmission Timeout)란?
패킷 유실 후 재전송이 일어나고, **재전송에 대한 ACK를 얼마나 기다려야 하는지에 대한 값**을 의미한다. <br>
보통 운영체제마다 별도의 초기값을 가지지만 네트워크에 따라 동적으로 변경되는데 이때 **RTT**에 의해 변한다. <br>
<br>

### ❓ RTT(Round Trip Time)란?
호스트 간 송신에 대한 응답(ACK)을 받기까지 걸리는 시간

#### ❓ sampleRTT?






