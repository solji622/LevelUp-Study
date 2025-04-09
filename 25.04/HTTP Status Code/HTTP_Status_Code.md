# HTTP 상태 코드 (HTTP Status Code)
<br>
<br>

## 📌 HTTP 상태 코드란?
특정 HTTP 요청의 성공 유무를 서버에서 알려주는 숫자 코드 <br>
<br>
![개발자 도구 네트워크탭](https://github.com/solji622/LevelUp-Study/blob/bf5dbf52c3ec2b66790d9cca0eddf24e2fd86ff5/25.04/HTTP%20Status%20Code/asset/chromeNetwork.png)
<br>
크롬 개발자 도구의 네트워크 탭을 보면 사진처럼 숫자 코드로 요청 결과를 간략하게 보여주기도 한다 <br>

3자리 숫자로 이루어져 있으며 총 다섯가지로 나누어진다
> **1XX** : 정보 (Informational) <br>
> **2XX** : 성공 (Success) <br>
> **3XX** : 리다이렉션 (Redirection) <br>
> **4XX** : 클라이언트 에러 (Client Error) <br>
> **5XX** : 서버 에러 (Server Error) <br>

<br>
<br>
<br>

## 📌 1XX
> 정보 제공, 요청이 수신되어 처리 중 <br>
현업에서도 잘 사용되지 않는 상태코드이다. <br>

|status code|message|desc|
|:---|:---|:---|
|100|Continue|요청이 처리 됨|
|101|Switching Protocol|서버의 프로토콜 전환|
|102|Processing|요청 처리 중, 제대로 된 응답 불가|
|103|Early Hints|웹페이지 리소스 사전 로드|

<br>
<Br>
<br>

## 📌 2XX
> 성공, 요청이 정상적으로 처리 되었다는 의미다 <br>
표에 있는 것 외에도 203, 206-8, 218, 226이 존재한다 <br>

|status code|message|desc|
|:---|:---|:---|
|200|Ok|요청이 정상적으로 처리됨|
|201|Created|요청 처리가 완료되었고 새 리소스가 생성됨|
|202|Accepted|요청은 받았으나 크고 무거워 아직 처리가 완료되지 않음|
|204|No Content|정상적인 요청이나 제공할 콘텐츠가 존재하지 않음|
|205|Reset Content|요청을 보낸 뷰를 리셋, 브라우저 새로고침하기|
