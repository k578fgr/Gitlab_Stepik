```
variables:
  ENV: "dev"
  DB_HOST_PROD: "db.prod.com"
  DB_HOST_DEV: "db.dev.com"

stages:
  - connect-db

connect-db-job:
  stage: "connect-db"
  rules:
  - if: '$CI_COMMIT_BRANCH == "main"'
    variables:
      ENV: "PROD"
  script:
    - echo "Подключаемся к $(eval echo \$DB_HOST_${ENV})."
```


Удивительно как я не понял, что можно так делать