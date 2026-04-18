```
variables:
  APP_NAME: "my_app"       # Название приложения
  DEPLOY_ENV: "staging"    # Окружение для деплоя
  VERSION: "1.0.0"         # Версия приложения


# Этапы пайплайна
stages:
  - hello

# Задание для тестирования
hello:
  stage: hello
  script:
    - echo Starting ${APP_NAME}, version:${VERSION}
```

Вывод:
Starting MyApp, version: 1.0.0