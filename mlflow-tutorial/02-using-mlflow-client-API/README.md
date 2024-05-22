# MLflow Tracking Client
- `MLFLOW_TRACKING_URI`를 따로 설정하지 않으면 `mlflow`를 실행한 디렉토리에 모델 관련된 attribute들이 저장된다.
    - experiments, models, etc
- `mlflow server`에 연결하기 위해서는 앞서 사용한 argment(`--host`, `--port`)를 조합해서 client에 넘겨준다.

    ```shell
    # bash 명령
    mlflow server --host 127.0.0.1 --port 8080
    ```
    ```python
    # python
    client = MlflowClient(tracking_uri="http://127.0.0.1:8080")`
    ```

# The Default Experiment
- 실험이 명시적으로 선언되지 않았을 때 사용하는 placeholder
- `mlflow.client.MlflowClient.search_experiments()`로 기본 구조 확인

    ```python
    import mlflow


    def assert_experiment_names_equal(experiments, expected_names):
        actual_names = [e.name for e in experiments if e.name != "Default"]
        assert actual_names == expected_names, (actual_names, expected_names)


    mlflow.set_tracking_uri("sqlite:///:memory:")
    client = mlflow.MlflowClient()

    # Create experiments
    for name, tags in [
        ("a", None),
        ("b", None),
        ("ab", {"k": "v"}),
        ("bb", {"k": "V"}),
    ]:
        client.create_experiment(name, tags=tags)

    # Search for experiments with name "a"
    experiments = client.search_experiments(filter_string="name = 'a'")
    assert_experiment_names_equal(experiments, ["a"])

    # Search for experiments with name starting with "a"
    experiments = client.search_experiments(filter_string="name LIKE 'a%'")
    assert_experiment_names_equal(experiments, ["ab", "a"])

    # Search for experiments with tag key "k" and value ending with "v" or "V"
    experiments = client.search_experiments(filter_string="tags.k ILIKE '%v'")
    assert_experiment_names_equal(experiments, ["bb", "ab"])

    # Search for experiments with name ending with "b" and tag {"k": "v"}
    experiments = client.search_experiments(filter_string="name LIKE '%b' AND tags.k = 'v'")
    assert_experiment_names_equal(experiments, ["ab"])

    # Sort experiments by name in ascending order
    experiments = client.search_experiments(order_by=["name"])
    assert_experiment_names_equal(experiments, ["a", "ab", "b", "bb"])

    # Sort experiments by ID in descending order
    experiments = client.search_experiments(order_by=["experiment_id DESC"])
    assert_experiment_names_equal(experiments, ["bb", "ab", "b", "a"])

    ```

# Searching Experiments
- `mlflow server`에 있는 실험과 연관된 metadata attributes 확인
    > metadata의 attributes를 기억하고 있는 것이 매우 중요

    ```python
    all_experiments = client.search_experiments()
    print(all_experiments)

    # 결과: Experiment 객체의 리스트
    [
        <Experiment: artifact_location='./mlruns/0',
        creation_time=None,
        experiment_id='0',
        last_update_time=None,
        lifecycle_stage='active',
        name='Default',
        tags={}>
    ]
    ```

- `name`, `lifecycle_stage`를 추출하는 예제
    ```python
    default_experiment = [
        {"name": experiment.name, "lifecycle_stage": experiment.lifecycle_stage}
        for experiment in all_experiments
        if experiment.name == "Default"
    ][0]

    pprint(default_experiment)

    ```