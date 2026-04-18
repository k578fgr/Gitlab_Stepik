
```
stages:
  - build
  - test
  - deploy
  - report

build_job:
  stage: build
  script:
    - echo "Build completed" > build.txt
  artifacts:
    paths:
      - build.txt

test_job:
  stage: test
  needs: ["build_job"]  # Зависит от build
  script:
    - echo "Tests passed" > test.txt
  artifacts:
    paths:
      - test.txt

deploy_job:
  stage: deploy
  needs: ["test_job"]  # Зависит от test 
  script:
    - echo "Deployed to staging" > deploy.txt
  artifacts:
    paths:
      - deploy.txt

report_job:
  stage: report
  needs: ["build_job", "test_job", "deploy_job"]  # Зависит от всех
  script:
    - cat build.txt test.txt deploy.txt >> final_report.txt
  artifacts:
    paths:
      - final_report.txt
```
