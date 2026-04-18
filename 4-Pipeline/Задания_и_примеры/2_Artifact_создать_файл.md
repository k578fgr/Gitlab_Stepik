### **Задание: Базовый Pipeline с артефактами (** `artifacts` )

**Цель:** Создать простой pipeline с тремя стадиями (`build`, `test`, `deploy`), где каждая стадия записывает данные в файл, который сохраняется как артефакт.

###  **Требования:**

1. **Стадии:**
    
    - `build` → `test` → `deploy`
        
2. **Действия:**
    
    - В каждой стадии создать текстовый файл (`build.txt`, `test.txt`, `deploy.txt`) с записью:
        
        ```bash
        echo "Running [STAGE_NAME] stage at $(date)" >> [STAGE_NAME].txt # [STAGE_NAME - тут название стадии build.txt или так test.txt
        ```
        
    - Сохранить файлы как **артефакты**, чтобы их можно было скачать после выполнения.
        
3. **Дополнительно:**
    
    - В конце `deploy-job` добавить сообщение `"Pipeline completed!"` в файл `summary.txt` и сохранить его.
      
      
      

Все файлы сохранились
Не забывать только создавать путь
Графа артефакт, позволяет их сохранять
variables в скобках - ${CI_JOB_STAGE}

```
stages:
  - build
  - test
  - deploy

build-job:
  stage: build
  script:
    - mkdir -p file
    - touch '$CI_JOB_STAGE'.txt
    - echo "Running '$CI_JOB_STAGE' stage at $(date)" >> "file/${CI_JOB_STAGE}.txt"
  artifacts:
    name: "$CI_JOB_STAGE"
    expire_in: 2 days
    paths:
      - file/

test-job:
  stage: test
  script:
    - touch ${CI_JOB_STAGE}.txt
    - echo "Running ${CI_JOB_STAGE} stage at $(date)" >> "file/${CI_JOB_STAGE}.txt"
  artifacts:
    name: "$CI_JOB_STAGE"
    expire_in: 2 days
    paths:
      - file/

deploy-job:
  stage: deploy
  script:
    - touch ${CI_JOB_STAGE}.txt
    - echo "Running ${CI_JOB_STAGE} stage at $(date)" >> "file/${CI_JOB_STAGE}.txt"
  after_script:
    - touch summary.txt
    - echo "Pipeline completed!" > "file/summary.txt"
  artifacts:
    name: "$CI_JOB_STAGE"
    expire_in: 2 days
    paths:
      - file/
```
