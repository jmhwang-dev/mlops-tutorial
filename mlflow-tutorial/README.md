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

