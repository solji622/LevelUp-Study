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


## 📌 RTO(Retransmission Timeout)
패킷 유실 후 재전송이 일어나고, **재전송에 대한 ACK를 얼마나 기다려야 하는지에 대한 값**을 의미한다. <br>
보통 운영체제마다 별도의 초기값을 가지지만 네트워크에 따라 동적으로 변경되는데 이때 **RTT**에 의해 변한다. <br>
<br>

### ❓ sampleRTT(Round Trip Time)
호스트 간 송신에 대한 응답(ACK)을 받기까지 걸리는 시간을 측정하는 것
sampleRTT가 모여 RTT 값의 범위를 정하고 이를 기반으로 RTO를 설정한다.

<br>

### ❓ timeout은 어떻게 설정할 수 있을까?
#### 1. timeout < RTT
> 조금만 더 기다리면 ACK를 받을 수 있는데도 timeout 될 수도 있다 > 불필요한 재전송 잦아짐 <br>

#### 2. timeout > RTT
> 충분히 RTT의 가변적인 상황을 커버할 수 있겠지만, 세그먼트 손실 시 처리가 느려질 수 있다 <br>
<br>

![timeout](https://github.com/solji622/LevelUp-Study/blob/a73e0aa1cb7b58021e74f726ae4860b82eb2600b/25.04/TCP%20Retransmission/asset/RTT_timeout.jpg)

<br>

RTT보다는 길게 설정해야 그나마 안정적이겠지만, TCP는 인터넷 상에서 서비스가 되기에 <br>
네트워크 상황에 따라 RTT가 매우 상이하고 불규칙적이다. <br>

따라서 TCP는 sampleRTT의 평균 값인 **Estimated RTT**를 계산한다. <br>
새로운 sampleRTT를 획득하자마자 TCP는 아래 공식에 따라 Estimated RTT를 계산한다. <br>
<br>
**`EstimatedRTT = (1 - α) * EstimatedRTT + α * SampleRTT`**
<br><br>
이 공식은 지수 가중 이동 평균(Exponential Weighted Moving Average) 방식으로 RTT를 계속 보정해가며 안정적인 RTO를 계산한다. <br>
α(알파)는 가중치 계수로 **작을 수록 과거의 값에 더 의존**하고, **클수록 최신 측정값(sampleRTT)에 민감하게 반응**한다. <br>
<br>
<br>

그러나 이 식은 **평균값**을 기반으로 하기에 통계적으로 **극단적인 값**에 영향을 받기 쉬워진다. <br>
> 예를 들어서 네트워크 지연으로 인해 sampleRTT가 평균 100ms인데 800ms가 측정되었다면 <br>
> 이 값이 EstimatedRTT를 확 끌어올려버리기 때문에 다음 RTO 설정이 과하게 커질 수 있고 <br>
> 불필요하게 응답을 오래 기다리게 된다. <br>

<br>
<br>

때문에 추가적으로 RTT의 변동성까지 고려하여 **편차와 함께 계산**하는 방식이 나타났다. <br>
<br>
**`DevRTT = (1 - β) * DevRTT + β * |SampleRTT - EstimatedRTT|`**
<br><br>
이 공식도 Estimated RTT 계산 공식과 동일한 방식이며 β(베타)는 알파와 똑같이 가중치 계수이다. <br>
`|SampleRTT - EstimatedRTT|`는 예측과 실제 간의 절댓값 차이, 오차를 의미하며 <br>
RTT가 예측값과 **많이 다르면, DevRTT가 커지고** RTT가 **안정적이라면, DevRTT는 작게 유지**된다. <br>
<br>
만약 **DevRTT가 크면 TCP는 RTT 예측이 불안정하다고 판단**하고, <br>
그에 따라 RTO를 더 여유 있게 설정하게 된다. <br>
<br>
***
<br>

### 💡 최종적으로 timeout값은 어떻게 정할까?
**`TimeoutInterval = EstimatedRTT + 4 * DevRTT`** <br>
<br>
`4 * DevRTT`를 더하는 이유는 **RTT의 큰 변동에도 안정적으로 대응**하기 위해서다. <br>
즉, 네트워크 상황에 문제가 생겨도 TCP가 성급하게 재전송하지 않도록 여유를 주는 것이다. <br>

<br>
<br>
<br>
<br>

## 📌 Wireshark로 보는 TCP 재전송 예제
> #### Wireshark란?
> 대표적인 패킷 캡쳐 프로그램, 네트워크에서 송수신되는 패킷을 모니터링 및 분석한다
<br>

![wireshark](https://github.com/solji622/LevelUp-Study/blob/0f17ee8c5e73cbd396004097cba38152c44eb1a0/25.04/TCP%20Retransmission/asset/wireshark.png)
1. 통신 방향은 10.10.10.1 → 192.168.0.1 <br>
2. 3번 패킷에서 `TCP Previous segment not captured` 메시지가 발생 → 패킷 유실 <br>
3. 이후 수신 측에서 중복된 ACK(Dup ACK) 보내기 시작 <br>
4. 3개의 Dup ACK 이후 송신 측의 빠른 재전송(Fast Retransmission) 발생 <br>

<br>

### ❓ Dup ACK & Fast Retransmit
**Duplicate ACK**, 중복된 ACK로 수신 측에서 이미 받은 패킷에 대하여 Ack를 한번 더 보낸다. <br>
이 Dup ACK가 3회 반복될 경우 재전송을 보내게 되는데, **빠른 재전송(Fast Retransmit)** 을 통하여 세그먼트를 **즉시** 전송해준다. <br>



