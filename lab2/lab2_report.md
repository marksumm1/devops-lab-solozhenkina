# Лабораторная работа №2

## CI/CD для Docker приложения

**Университет:** Университет ИТМО  
**Группа:** U4225  
**Автор:** Соложенкина Елизавета  

## Цель работы

Настроить CI/CD-пайплайн с использованием GitHub Actions для автоматической сборки Docker-образа, его публикации в Docker Hub и выполнения шага деплоя при изменении кода в ветке `main`.

## 1. Подготовка проекта

Для лабораторной работы в каталоге `lab2` были созданы файлы:

- `app.py` — Flask-приложение;
- `requirements.txt` — список зависимостей;
- `Dockerfile` — инструкция для сборки Docker-образа.

Содержимое `app.py`:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello from Docker CI/CD!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

Содержимое `requirements.txt`:

```text
Flask==3.0.3
```

Содержимое `Dockerfile`:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

![Структура файлов проекта](images/01_project_structure.png)

**Рисунок 1 — Структура файлов приложения для лабораторной работы №2**

## 2. Создание репозитория в Docker Hub

В Docker Hub был создан публичный репозиторий:

```text
docksumm/my-flask-app
```

Он используется для хранения Docker-образа приложения.

![Репозиторий Docker Hub](images/02_dockerhub_repository.png)

**Рисунок 2 — Созданный репозиторий `docksumm/my-flask-app` в Docker Hub**

## 3. Настройка секретов GitHub

Для авторизации GitHub Actions в Docker Hub в настройках GitHub-репозитория были добавлены два Repository secrets:

- `DOCKER_USERNAME` — имя пользователя Docker Hub;
- `DOCKER_PASSWORD` — Personal Access Token Docker Hub.

Значения секретов скрыты и не хранятся в открытом виде в workflow.

![Секреты GitHub Actions](images/03_github_secrets.png)

**Рисунок 3 — Настроенные секреты Docker Hub в GitHub Actions**

## 4. Настройка CI/CD-пайплайна

В корне репозитория был создан файл:

```text
.github/workflows/docker-build.yml
```

Пайплайн запускается при push в ветку `main`, использует Ubuntu runner, выполняет checkout кода, настраивает Docker Buildx, авторизуется в Docker Hub, собирает и публикует образ, после чего выполняет шаг деплоя.

Конфигурация workflow:

```yaml
name: Docker CI/CD

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: ./lab2
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/my-flask-app:latest

      - name: Deploy
        run: echo "Deploying application..."
```

Параметр `context: ./lab2` указывает GitHub Actions использовать `Dockerfile` и остальные файлы приложения из каталога `lab2`.

![Конфигурация workflow](images/04_workflow_config.png)

**Рисунок 4 — Конфигурация CI/CD-пайплайна GitHub Actions**

## 5. Проверка выполнения GitHub Actions

После добавления workflow был выполнен push в ветку `main`. GitHub Actions автоматически запустил workflow `Docker CI/CD`.

Запуск завершился со статусом `Success`.

![Успешный запуск workflow](images/05_workflow_success.png)

**Рисунок 5 — Успешное выполнение workflow `Docker CI/CD`**

В журнале выполнения видно успешное прохождение основных этапов:

- `Checkout code`;
- `Set up Docker Buildx`;
- `Log in to Docker Hub`;
- `Build and push Docker image`;
- `Deploy`.

![Этапы workflow](images/06_workflow_steps.png)

**Рисунок 6 — Успешное выполнение этапов CI/CD-пайплайна**

## 6. Проверка результата в Docker Hub

После успешного выполнения GitHub Actions в репозитории `docksumm/my-flask-app` появился образ с тегом:

```text
latest
```

Это подтверждает, что Docker-образ был автоматически собран и опубликован в Docker Hub.

![Docker Hub latest](images/07_dockerhub_latest.png)

**Рисунок 7 — Docker-образ с тегом `latest` в Docker Hub**

Образ можно загрузить командой:

```bash
docker pull docksumm/my-flask-app:latest
```

## Вывод

В ходе лабораторной работы был настроен CI/CD-пайплайн на основе GitHub Actions. При push в ветку `main` автоматически запускается workflow, который получает исходный код, настраивает Docker Buildx, авторизуется в Docker Hub через GitHub Secrets, собирает Docker-образ и публикует его с тегом `latest`. Также в пайплайн добавлен демонстрационный шаг деплоя. Успешный статус GitHub Actions и появление образа в Docker Hub подтверждают корректность настройки.
