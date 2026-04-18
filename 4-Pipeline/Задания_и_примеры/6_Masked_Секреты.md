**Задача:**

1. Создать в GitLab (Settings → CI/CD → Variables) переменную `API_KEY` со значением `secret123` и включить **Masked**.
2. Написать job `show-key`, который выводит значение `API_KEY` (в логах должно отображаться `*****`).
   
```
stages:
  - build

build-job:
  stage: build
  variables:
    ENV: "dev"
  script:
    - echo "API_KEY -" ${API_KEY}
```

