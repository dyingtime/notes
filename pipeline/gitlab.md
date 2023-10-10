## jenkins pipeline


### how to pass variable to downstream jobs
```yaml
stages:
  - stage1
  - stage2

stage1:
  stage: stage1
  image: ubuntu:latest
  script:
    - export TEST_ENV=123
    - echo "TEST_ENV=TEST_ENV" >> build.env
  artifacts:
    reports:
      dotenv: build.env

stage2:
  stage: stage2
  trigger:
    forward:
      pipeline_variables: true
    project: 'downstream'
    branch: 'main'
    strategy: depend
  variables:
    TEST_ENV: $TEST_ENV
  rules:
    - if: $BACKWARD_COMPATIBILITY == "true"
```