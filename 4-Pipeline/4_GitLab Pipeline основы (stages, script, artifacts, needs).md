# Чтобы отредактировать пайп
Заходим в Pipeline editor

![alt text](/image_folder/image-17.png)

Стадии того, за что отвечает джоб
стадии с точкой, не входят в основную джобу и делают что-то малозначительное перед сборкой и после деплоя. Хотя бы один джоб должен быть между них
```
- .pre
- build
- test
- deploy
- .post
```

Задействовать .pre начало
![alt text](/image_folder/image-18.png)



![alt text](/image_folder/image-19.png)

![alt text](/image_folder/image-20.png)

## After_script

![alt text](/image_folder/image-21.png)


## Использовать переменные

![alt text](/image_folder/image-22.png)

![alt text](/image_folder/image-23.png)

## Отношение между Jobs - Needs

![alt text](/image_folder/image-24.png)

![alt text](/image_folder/image-25.png)

![alt text](/image_folder/image-26.png)

## Paths

## 1. В секции `cache: paths`


```
cache:
  paths:
    - node_modules/
    - target/
    - .cache/
  key: "$CI_COMMIT_REF_SLUG"
```

**Назначение**: Указывает какие файлы/директории кешировать между пайплайнами


## 2. В секции `artifacts: paths`

yaml


```
test:
  script: 
    - npm test
  artifacts:
    paths:
      - coverage/
      - test-results.xml
    when: always
```


**Назначение**: Определяет какие файлы сохранить как артефакты

## 3. В правилах `rules: changes`


```
deploy:
  script: 
    - ./deploy.sh
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      changes:
        - docker-compose.yml
        - deployment/**/*
      when: manual
```


**Назначение**: Запускать job только при изменениях в указанных путях

![alt text](/image_folder/image-27.png)

## Чтобы вывелась ошибка

```
job-for-fail:
  script:
    - prd & echo "1"
    - echo $?
    - (echo "2" && pwd")
    - echo $?
```

В первом случае ошибки не будет, а во втором уже да
Поэтому в гитлаб нужно заключать в скобки условия и амперсанты

## Пример before_script


```
job1:
  script:
    - echo "Job 1"

job2
  script:
    - echo "Job 2"

job3:
  before_script:
    - echo "[job_before_script] Execute this command before any *script* "
  script:
    - echo "Job 3"
  after_script:
    - echo "[job_before_script]" Execute this command after the *script* ""
```


## вводим ручками пример

Пример 1

```
stages:
  - build
    
job-name:
  stage: build
  script:
    - echo: "Building"
```

Пример 2
```
stages:
  - .pre
  - build
  - test
  - deploy
  - .post
    
build_app:
  stage: build
  script: echo "Building"
  
test_app:
  stage: test
  script: echo "Run test"
  
deploy_app:
  stage: deploy
  script:
    - echo "start deploy"
    - echo "job is done"
```


## Needs для чего вообще нужен

### **Основное назначение**

`needs` создаёт зависимости между джобами и позволяет им запускаться раньше, без ожидания завершения всей предыдущей стадии.

---

### **Преимущества использования `needs`**

#### **1. Ускорение пайплайнов**


```yaml
stages:
  - build
  - test
  - deploy

build_job:
  stage: build
  script: echo "Building..."

unit_test:
  stage: test
  script: echo "Unit testing"
  needs: ["build_job"]  # Запустится сразу после build_job, не ждя всю стадию build

integration_test:
  stage: test
  script: echo "Integration testing"
  needs: ["build_job"]  # Также запустится сразу после build_job
```


#### **2. Параллельное выполнение**

yaml


```
lint:
  stage: test
  script: echo "Linting"

unit_test:
  stage: test  
  script: echo "Unit tests"
  needs: []  # Запустится сразу, без зависимостей

integration_test:
  stage: test
  script: echo "Integration tests"
  needs: ["lint", "unit_test"]  # Ждёт завершения lint и unit_test
```


---

### **Типы использования**

#### **1. Зависимость от конкретной джобы**

```
build:app:
  stage: build
  script: echo "Build app"

build:docs:
  stage: build  
  script: echo "Build docs"

test:app:
  stage: test
  needs: ["build:app"]  # Только от build:app
  script: echo "Test app"
```


#### **2. Зависимость от артефактов**

```

build:
  stage: build
  script: echo "Building..."
  artifacts:
    paths:
      - build/

test:
  stage: test
  needs: ["build"]  # Получает артефакты от build
  script: echo "Testing with artifacts"
```


#### **3. Зависимость от джоб в других стадиях**

```
stages:
  - build
  - test
  - deploy

build:backend:
  stage: build
  script: echo "Build backend"

build:frontend:
  stage: build
  script: echo "Build frontend"

test:backend:
  stage: test
  needs: ["build:backend"]  # Запустится сразу после build:backend

deploy:staging:
  stage: deploy
  needs: ["build:backend", "build:frontend"]  # Ждёт оба build
```


---

### **Ограничения и особенности**

#### **Визуализация в GitLab UI**

- Пайплайн отображается как ориентированный граф
    
- Видны прямые зависимости между джобами
    

#### **Ограничения:**

- `needs` не может ссылаться на джобы из последующих стадий
    
- Максимум 5 джоб в одном `needs` (можно увеличить в настройках)
    
- Требует GitLab 12.2+
    

---

### **Практический пример**

```
stages:
  - build
  - test
  - deploy

build:app:
  stage: build
  script: ./build-app.sh

build:image:
  stage: build
  script: ./build-image.sh

test:unit:
  stage: test
  needs: ["build:app"]
  script: ./run-unit-tests.sh

test:integration:
  stage: test
  needs: ["build:app", "build:image"]
  script: ./run-integration-tests.sh

deploy:
  stage: deploy
  needs: ["test:unit", "test:integration"]
  script: ./deploy.sh
```


**Результат**: `test:unit` и `test:integration` запустятся параллельно сразу после завершения соответствующих build-джоб, что значительно ускорит общее время выполнения пайплайна.
