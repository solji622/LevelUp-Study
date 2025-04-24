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

<br>
<br>
<br>
<br>

## 📌 TCP 재전송과 혼잡 제어의 관계
TCP에는 크게 흐름/오류/혼잡 3가지의 제어 기능이 존재한다.
1. 흐름 제어 : 송수신 측 사이에서 전송되는 데이터 양 조절
2. 오류 제어 : 데이터가 유실되거나 잘못된 데이터 수신될 경우 대처
3. 혼잡 제어 : 네트워크 혼잡 대처 (송신 측의 데이터 전송 속도 제어) 

<br>
<br>

### ❓ TCP 재전송은 왜 혼잡일까?
> TCP 재전송의 원인 중 하나인 패킷 유실은 TCP가 망의 혼잡에 기인한 원인으로 판단하기 때문이다.

회피 방식들을 활용한 다양한 혼잡 제어 알고리즘들이 존재하고, <br>
알고리즘들은 혼잡 회피 방식들을 적절히 섞어서 사용하되 혼잡 발생 시 어떻게 대처하는지에 따라 나뉜다고 한다. <br>
이 중 빠른 재전송(Fast Recovery)이 혼잡 회피 방식 중 하나이라 볼 수 있다. <br>

<br>
<br>

### 💡 혼잡 회피 방법
#### 1. AIMD
Additive Increase / Multicative Decrease, 합 증가 / 곱 감소 <br>
네트워크에 별 문제가 없어서 전송 속도를 빠르게 하고 싶다면 **혼잡 윈도우 크기를 1씩 증가**, <br>
만약에 패킷 전송 실패 또는 time_out 와 같은 혼잡 상태 감지 시 **혼잡 윈도우 크기를 반으로 줄인다.** <br>
간단하지만 공평한 방식으로, 여러 호스트가 한 네트워크를 공유 중일 때 나중에 진입하는 쪽이 <br>
처음엔 작은 혼잡 윈도우를 가지게 되어 불리하지만, 시간이 흐를수록 **평형 상태로 수렴**하게 된다. <br>

<br>

> **혼잡 윈도우란?** <br>
> 송신측이 ACK를 받기 전에 보낼 수 있는 패킷의 최대 개수를 의미한다.

<br>

그러나 문제점이 존재한다. 네트워크 대역이 여유로움에도 불구하고 윈도우 크기를 점진적으로 늘리면서 접근하기에 <br>
초기 네트워크의 큰 대역폭을 바로 사용하지 못하고, <br>
네트워크가 혼잡해지는 상황을 미리 감지하지 못해서 혼잡해진 뒤에야 대역폭을 줄이게 된다. <br>
때문에 모든 대역을 활용하여 제대로 된 속도로 통신하기까지의 시간이 꽤 걸리는 편이다. <br>

<br>

#### 2. Slow Start
느린 시작, 혼잡 상황 발생 전에도 제대로 된 속도를 내지 못하는 AIMD에 비해 <br>
**이름과는 반대**로 시작부터 빠르게 윈도우를 증가시키고 특정 시기가 오면 윈도우를 확 줄여버린다. <br>
물론 AIMD와 동일하게 처음엔 패킷을 하나씩 전송하지만, 문제없이 도착하면 <br>
윈도우 사이즈를 **두배씩 증가**시키면서 빠르게 늘리고 혼잡 현상이 발생하면 윈도우 사이즈를 **1로 확 줄여버린다.** <br>
AIMD보다 효율적으로 동작하나, 처음에 하나씩 전송하는 것 때문에 전송 속도를 올리는 데 시간이 오래 걸린다는 단점이 있다. <br>

<br>

#### 3. 빠른 재전송 (Fast Retransmit)
수신 측에서 패킷을 받을 때 먼저 도착해야 할 패킷이 도착하지 않고 다음 패킷이 도착한 경우에도 ACK를 보낸다. <br>
단, 순서대로 잘 도착한 마지막 패킷의 다음 패킷의 순번을 ACK 패킷에 실어보낼 때 중간에 패킷 하나가 손실되면
**송신 측에서는 순번이 중복된 ACK 패킷을 받게 된다.** 이것을 감지하게 되면 문제가 되는 순번의 패킷을 재전송할 수 있는데
빠른 재전송은 이 중복된 순번의 패킷을 **3개 (3 ACK Duplicated) 받게 될 때 재전송**해주는 것이다. <br>
그리고 약간의 혼잡이 발생한 것으로 간주하여 **윈도우 사이즈를 절반으로 줄여버린다.** <br>

<br>

#### 4. 빠른 회복 (Fast Recovery)
혼잡 상태가 되면 윈도우 사이즈를 절반으로 줄이고 **선형으로 증가시킨다.** <br>
이 방법까지 적용하면 혼잡 상황을 한번 겪고 난 뒤 순수한 AIMD 방식으로 동작하게 된다. <br>

<br>
<br>

### 💡 혼잡 제어 알고리즘
#### 1. TCP Tehoe
혼잡 제어의 초기 정책, 빠른 재전송이 처음으로 도입된 정책이다. <br>
처음엔 동일하게 slow start 방식으로 윈도우를 증가시키가다가 ssthresh 시점 이후부터 AIMD 방식을 사용한다. <br>

> **ssthresh란?** <br>
> slow start threshold, 여기까지만 slow start를 사용하겠다. <br>
> 특정 임계점을 윈도우 사이즈가 넘어가면 이후부터 AIMD 방식을 사용하여 윈도우를 증가한다. <br>

<img src="https://github.com/solji622/LevelUp-Study/blob/70ac9fb4bc04db1470ceffbd4e01c0cd91772e00/25.04/TCP%20Retransmission/asset/tahoe-ssthresh.png" width="70%">

검정색 선이 ssthresh로, Timeout이나 3 ACK가 발생할 때마다 **윈도우는 1로,** <br>
ssthresh는 이전 혼잡 상황 발생 시 **윈도우 크기의 절반으로** 줄어든다. <br>
하지만 혼잡 상황 발생 시마다 윈도우를 1로 초기화해 다시 증가시키는 부분이 비효율적이다. <br>
이러한 점을 개선 시키기 위해 **빠른 회복을 적용한 TCP Reno**가 존재한다. <br>

<br>

#### 2. TCP Reno
Tahoe와 동일하지만 3 ACK와 Timeout을 **구분해서 대응**하는 점에서 차이가 난다. <br>
3 ACK 발생 시 윈도우 크기를 1로 초기화하지 않고 **AIMD처럼 윈도우 크기를 절반**으로 줄이며 <br>
ssthresh 값 역시 **줄어든 윈도우 값으로 설정**한다. <br>
Timeout 발생 시 Tahoe와 동일하게 사이즈를 1로 줄이고 Slow start 를 시작하지만 **ssthresh 값은 변경되지 않는다.** <br>

<img src="https://github.com/solji622/LevelUp-Study/blob/3a222428d7ae9539c30aed127fad62178836ec78/25.04/TCP%20Retransmission/asset/reno-ssthresh.png" width="70%">

<br>
<br>
<br>

***
<br>
<br>

## 👾 결론적으로
TCP의 재전송은 단순히 패킷을 다시 보내는 기술이 아닌 망의 상태를 파악하고 <br>
네트워크 혼잡을 완화하고 네트워크 전체의 안정성과 성능을 유지하려는 핵심 기능으로 이해할 수 있다. <br>

