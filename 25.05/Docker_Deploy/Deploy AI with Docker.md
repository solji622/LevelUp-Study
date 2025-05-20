# Docker로 AI 모델 배포하기
<br>
<br>

## 📌 Docker란?
컨테이너를 사용해서 각각의 프로그램을 **분리된 환경**에서 실행 및 관리가 가능한 툴

#### ▪️ 컨테이너(Container)
> Docker는 하나의 컴퓨터 환경 내에서 여러 개의 미니 컴퓨터 환경 구성이 가능한데 <br>
> 여기서 **'미니 컴퓨터'** 의 역할을 하는 것이다. <br>
> 미니 컴퓨터는 어떤 환경에서나 실행이 가능하게 **필요한 모든 요소를 포함하는 소프트웨어 패키지**라 볼 수 있다.

<br>
<br>

## 📌 Docker는 왜 사용할까?
> **이식성** (특정 프로그램을 다른 곳으로 쉽게 옴겨서 설치 및 실행이 가능한 특성) 이 존재하기 때문!

1. 매번 설치 과정 일일이 거치지 않아도 됨
2. 항상 일관된 프로그램 설치
3. 독립적인 환경에서 실행하므로 프로그램 간 충돌 X


<br>
<br>
<br>
<br>

## 💡 AI 모델 배포하기 - HuggingFace 사용
### 📌 HuggingFace(허깅페이스)란?
이름 자체로는 AI가 인간을 돕는 친근한 존재가 되기를 바라는 의미를 가지고 있으며, <br>
트랜스포머 기반으로 자연어 처리(NLP)와 머신러닝 모델을 개발, 훈련, 배포할 수 있도록 돕는 **오픈소스 커뮤니티 플랫폼**이다. <br>
> **트랜스포머(Transformers)?** <br>
> 단어나 문장과 같은 입력 데이터에서 중요한 정보를 추출하고 **출력 데이터를 생성하는 딥러닝 모델**

<br>
배포를 위해 AI 모델을 Flask를 통해 API로 감쌀 것이다. <br>
모델 자체는 Python에서 돌릴 수 있긴 하나, API로 감싸면 어떤 환경에서도 <br>
쉽게 요청해서 사용이 가능해진다! <br>
<br>

<br>

### 👾 API (Flask + HuggingFace)
``` python
from flask import Flask, request, jsonify
from transformers import pipeline

app = Flask(__name__) # Flask 앱 객체 생성
classifier = pipeline("sentiment-analysis")  # 감정 분석 모델 로딩

@app.route("/predict", methods=["POST"])
def predict():
    text = request.json["text"]
    result = classifier(text)
    return jsonify(result)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000) # 모든 IP 접속 가능 (== 0.0.0.0)
```
<br>

**`classifier = pipeline("sentiment-analysis")` : 모델 로딩, 물론 다른 모델도 가능** <br>

**1.** 트랜스포머 파이프라인의 태스크명을 변경하여 사용하기 <br>
  &nbsp; : 텍스트 분류(text-classification), 개체명 인식(ner), 질문 답변(question-answering), 번역(translation) 등 다양한 태스크 존재 <br>

**2.** 허깅페이스 허브의 특정 모델 이름 직접 지정하기 (불러오기) <br>
   &nbsp; : 파이프라인에 `model=` 인자를 넣어 모델 불러오기 가능


<br>

**`def predict()` : URL 경로를 POST 요청으로 받을 때 실행되는 함수 정의** <br>

1. 클라이언트가 보낸 JSON 데이터에서 "text" 키의 값을 꺼내서 text 변수에 저장
2. 감정 분석 모델에 text 넣고, 결과를 result에 저장
3. JSON 형태로 결과 변환 후 클라이언트에 응답 <br> 

<br>
<br>

### 👾 Dockerfile
> Docker 이미지를 만들 때 활용하는 것, 즉 이미지를 만들게 해주는 파일.

``` dockerfile
FROM python:3.9
WORKDIR /app
COPY . . 
RUN pip install flask transformers torch # 이미지 생성 시 flask 트랜스포머 다운로드
EXPOSE 5000 
CMD ["python", "app.py"]
```

<br>
<br>

### 👾 이미지 빌드하여 컨테이너 실행하기
```bash
docker build -t ai-flask-api . # 이미지 빌드
docker run -p 5000:5000 ai-flask-api # 컨테이너 실행
```

<br>
<br>

### 👾 API 테스트 해보기
```bash
curl -X POST http://localhost:5000/predict \
     -H "Content-Type: application/json" \
     -d '{"text": "진짜 좋아요!"}'
```

결과로는 터미널에 JSON 형태로 표시된다.

<br>
<br>
<br>

***

<br>
<br>

## 💡 실제 웹 프로젝트에 적용한다면?
### 👾 REST API 호출
``` javascript
fetch("http://localhost:5000/predict", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ text: userInput }),
})
.then(response => response.json())
.then(data => {
  console.log(data); // 예측 결과 처리
})
.catch(error => {
  console.error("API 요청 실패:", error);
});

```

<br>
<br>

### 👾 입력 폼 구현
```html
<input id="textInput" type="text" placeholder="텍스트 입력">
<button onclick="sendText()">분석하기</button>
<div id="result"></div>

<script>
    function sendText() {
        const button = document.querySelector("button");
        button.disabled = true;
        document.getElementById("result").innerText = "분석 중...";
        
        const text = document.getElementById("textInput").value;
        fetch("http://localhost:5000/predict", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ text }),
        })
        .then(res => res.json())
        .then(data => {
            document.getElementById("result").innerText = JSON.stringify(data);
        })
        .catch(() => {
            document.getElementById("result").innerText = "서버 오류가 발생했습니다.";
        })
        .finally(() => {
            button.disabled = false;
        });
    }
</script>
```


