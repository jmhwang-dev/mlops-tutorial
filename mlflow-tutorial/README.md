# v1.20.2
## Tracking Server
- artifact 저장
- `mlflow ui` -> localhost:5000
    - run
    - artifact
    - model
- mlflow 저장 방식 (/mlruns)
    - mlruns/[id]/meta.yaml
    - `mlflow server`
        - `--backend-store-uri [PATH]`
        - `--default-artifact-root [URI]`

- mlflow를 사용한 serving 맛보기
    - serving: 해당 서버에 api를 보내서 `predict()`를 사용할 수 있게 만듬
    - /mlruns가 있는 경로에서 `mlflow models serve`
        - `-m [MODEL_PATH]`
        - `-p [PORT]`
    - `POST /invocations` API 요청 수행

        ```bash
        curl \
        -X POST \
        -H "Content-Type:application/json" \
        --data '{"columns": ["..", ".."], "data":[[.., ..], [.., ..]]}' \
        http://localhost:1234/invocations
        ```

- Automatic Logging
    - `mlflow.sklearn.autolog()`: sklearn 모델을 자동으로 로깅
    - 다양한 파이썬 패키지 지원

# Model Serving
- Serving?
    - ML Model을 서비스화하는 것
- 제공 방식
    - HTTP API Request
    - 챗봇 대화 등
- 필요이유?
    - 모델 개발 과정과 소프트웨어 개발 과정의 파편화
    - 모델 평가 방식 및 모니터링 구축 어려움
- Serving Framework 간편화 도구
    - Seldon Core
    - TFServing
    - KFServing
    - Touch Serve
    - BentoML

- PoC와 같은 초기 단계에서는 웹 프레임워크로 모델 코드를 감싼 후, API 서버 개발, docker, Load Balancer 구성으로 시작

## Flask
- 웹 앱 프레임워크
    - `app.py`
- Routing
    ```python
    # 예시
    app = Flask(__name__)
    @app.route("/predict", methods=["POST", "PUT"])
    def inference():
        return json.dumps({...}, 200)
    ```
    
    ```bash
    # 확인 예시
    curl -X POST http://localhost:5000/predict
    ```
- 실습
    ```python
    # 모델 서빙 서버
    app = Flask(__name__)
    model = pickle.load(open('model.pkl', 'rb'))

    @app.route("/predict", method=['POST'])
    def make_predict():
        request_body = request.get_json(force=True)
        
        # dataset 설정 생략 ...
        y_test = model.predict(X_test)
        response_body = jsonify(result=y_test.tolist())
        return response_body
    ```
    ```shell
    # 확인
    curl -X POST -H "Content-Type:application/json" \
    --data '
    {
        "sepal_length": 5.9,
        "sepal_width": 3.0,
        "petal_length": 5.1,
        "petal_width": 1.8
    }
    ' \
    http://localhost:5000/predict
    ```