🐳 Docker Best Practices Guide

    Комплексное руководство по работе с Docker, Docker Compose и Dockerfile для эффективной разработки и деплоя приложений.

📋 Содержание

    Docker Core Commands
    Docker Compose Guide
    Dockerfile Best Practices
    Security & Optimization
    Advanced Tips

🚀 Docker Core Commands
Управление контейнерами
Команда	Описание	Пример
docker create	Создать контейнер из образа	docker create --name my_redis --expose 6379 redis:6.2
docker start	Запустить созданный контейнер	docker start my_redis
docker stop	Остановить работающий контейнер	docker stop my_redis
docker ps	Показать работающие контейнеры	docker ps -a (все контейнеры)
docker exec	Выполнить команду в контейнере	docker exec -it my_redis redis-cli
docker logs	Просмотреть логи контейнера	docker logs -f my_redis
docker rm	Удалить остановленный контейнер	docker rm my_redis
Управление образами
Команда	Описание	Пример
docker images	Показать список образов	docker images -a (включая промежуточные)
docker build	Создать образ из Dockerfile	docker build -t myapp:1.0 .
docker pull	Загрузить образ из реестра	docker pull nginx:latest
docker push	Отправить образ в реестр	docker push myrepo/myapp:1.0
docker rmi	Удалить образ	docker rmi nginx:latest
docker tag	Создать тег для образа	docker tag myapp:1.0 myrepo/myapp:stable
Сеть и тома

# Создать сетьdocker network create my_network# Подключить контейнер к сетиdocker network connect my_network my_container# Создать томdocker volume create my_volume# Подключить том к контейнеруdocker run -v my_volume:/data my_image

 
📦 Docker Compose Guide 
Основные команды 
Команда
 
	
Описание
 
 
docker compose up	Создать и запустить сервисы 
docker compose down	Остановить и удалить сервисы 
docker compose start	Запустить уже созданные сервисы 
docker compose stop	Остановить сервисы 
docker compose restart	Перезапустить сервисы 
docker compose ps	Показать статус сервисов 
docker compose logs	Показать логи сервисов 
docker compose build	Пересобрать образы сервисов 
docker compose pull	Загрузить последние образы 
   
Пример docker-compose.yml 
yaml
 
  
 
version: '3.8'

services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/app
    environment:
      - FLASK_ENV=development
    depends_on:
      - redis

  redis:
    image: redis:6.2
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  redis_data:
 
 
 
Мульти-файловая конфигурация 
bash
 
  
 
# Использование нескольких compose-файлов
docker compose -f docker-compose.yml -f docker-compose.dev.yml up

# Объединение файлов из Git-репозитория
docker compose -f https://github.com/user/repo.git@main up

# Использование OCI-артефактов
docker compose -f oci://registry.example.com/my-compose:v1.0 up
 
 
 
  

Основной compose.yml

Перекрытие compose.dev.yml

Перекрытие compose.prod.yml

Финальная конфигурация Dev

Финальная конфигурация Prod
  
🔧 Dockerfile Best Practices 
1. Многоступенчатая сборка (Multi-stage Builds) 
dockerfile
 
  
 
# Стадия сборки
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Стадия выполнения
FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
CMD ["node", "dist/main.js"]
 
 
 
2. Выбор базового образа 
Тип образа
 
	
Когда использовать
 
	
Примеры
 
 
Alpine	Минимальный размер, простые приложения	node:18-alpine, python:3.11-alpine 
Debian Slim	Баланс размера и совместимости	node:18-slim, python:3.11-slim 
Ubuntu	Максимальная совместимость	ubuntu:22.04 
Distroless	Максимальная безопасность	gcr.io/distroless/nodejs18 
   

     

    💡 Совет: Используйте официальный образ с пометкой "Official" или "Verified Publisher" в Docker Hub для обеспечения безопасности и надежности. 
     

3. Оптимизация слоя кэша (Layer Caching) 
dockerfile
 
  
 
# ❌ Плохо - слои часто меняются
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install

# ✅ Хорошо - лучше использует кэш
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
 
 
 
4. Минимизация размеров образа 
dockerfile
 
  
 
# Используйте .dockerignore для исключения ненужных файлов
# node_modules
# npm-debug.log
# .git
# README.md

# Объединяйте команды RUN
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        git \
    && rm -rf /var/lib/apt/lists/*

# Используйте многоступенчатую сборку
COPY --from=builder /app/dist ./dist
 
 
 
5. Безопасность 
dockerfile
 
  
 
# Не запускайте от root
FROM node:18-alpine
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

# Используйте специфичные теги версий
FROM node:18.17.1-alpine

# Сканируйте образ на уязвимости
# docker scan myapp:1.0
 
 
 
Рекомендуемый порядок инструкций в Dockerfile 
dockerfile
 
  
 
# 1. Аргументы и базовый образ
ARG NODE_VERSION=18.17.1
FROM node:${NODE_VERSION}-alpine

# 2. Метаданные
LABEL maintainer="your-email@example.com"
LABEL version="1.0"

# 3. Переменные окружения
ENV NODE_ENV=production
ENV APP_HOME=/app

# 4. Рабочая директория
WORKDIR $APP_HOME

# 5. Копирование зависимостей
COPY package*.json ./

# 6. Установка зависимостей
RUN npm ci --only=production

# 7. Копирование кода
COPY . .

# 8. Настройка пользователя
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

# 9. Экспорт портов
EXPOSE 3000

# 10. Команда запуска
CMD ["node", "index.js"]
 
 
 
🔒 Security & Optimization 
Безопасность Dockerfile 
Практика
 
	
Описание
 
	
Пример
 
 
Минимизация привилегий	Не запускайте от root	USER nodejs 
Специфичные теги	Избегайте тега latest	FROM node:18.17.1-alpine 
Сканирование уязвимостей	Используйте docker scan	docker scan myapp:1.0 
Безопасный базовый образ	Используйте distroless или alpine	FROM gcr.io/distroless/nodejs18 
Секреты	Не храните секреты в Dockerfile	Используйте Docker secrets или переменные окружения 
   
Оптимизация производительности 
bash
 
  
 
# Используйте BuildKit для более быстрых сборок
DOCKER_BUILDKIT=1 docker build .

# Используйте кэш для сборки
docker build --cache-from myapp:previous -t myapp:new .

# Оптимизация слоев
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        package1 \
        package2 \
    && rm -rf /var/lib/apt/lists/*

# Используйте .dockerignore для исключения ненужных файлов
echo "node_modules" >> .dockerignore
echo ".git" >> .dockerignore
 
 
 
🎯 Advanced Tips 
1. Healthchecks 
dockerfile
 
  
 
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/ || exit 1
 
 
 
2. Настройка времени сборки 
dockerfile
 
  
 
# Используйте ARG для переменных сборки
ARG BUILD_DATE
ARG VCS_REF
LABEL org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.vcs-ref=$VCS_REF
 
 
 
3. Использование BuildKit 
bash
 
  
 
# Включите BuildKit в ~/.bashrc или ~/.zshrc
export DOCKER_BUILDKIT=1

# Используйте секреты в сборке
docker build --secret id=github_token,mytoken .
 
 
 
4. Multi-arch builds 
bash
 
  
 
# Сборка для нескольких архитектур
docker buildx create --use
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .
 
 
 
5. Оптимизация для CI/CD 
bash
 
  
 
# Используйте кэш для ускорения сборок в CI
docker build \
  --cache-from myapp:latest \
  --tag myapp:${GIT_COMMIT} \
  --tag myapp:latest \
  .

# Используйте Docker Build Action в GitHub Actions
- name: Build and push Docker image
  uses: docker/build-push-action@v4
  with:
    context: .
    push: true
    tags: user/app:latest
    cache-from: type=gha
    cache-to: type=gha,mode=max
 
 
 
📚 Дополнительные ресурсы 

     Docker Official Documentation 
     Dockerfile Reference 
     Docker Compose Reference 
     Docker Security Cheat Sheet 
     Best Practices for Building Docker Images 
     

     

    💡 Pro Tip: Регулярно обновляйте свои базовые образы и зависимости для обеспечения безопасности и получения последних исправлений ошибок. Используйте docker pull --pull для получения последних версий образов. 
     
