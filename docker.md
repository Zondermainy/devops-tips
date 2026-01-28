### DOCKER-COMPOSE
| Команда | Описание |
|---------|----------|
| `docker compose up` | Создать и запустить сервисы |
| `docker compose down` | Остановить и удалить сервисы |
| `docker compose start` | Запустить уже созданные сервисы |
| `docker compose stop` | Остановить сервисы |
| `docker compose restart` | Перезапустить сервисы |
| `docker compose ps` | Показать статус сервисов |
| `docker compose logs` | Показать логи сервисов |
| `docker compose build` | Пересобрать образы сервисов |
| `docker compose pull` | Загрузить последние образы |

## Пример манифеста docker-compose
```yaml
version: '3.8'              # Версия формата docker-compose
services:                   # Определение сервисов (контейнеров)
  nginx:                    # Имя сервиса - можно обращаться по этому имени внутри docker-compose сети
    image: nginx:latest     # Официальный образ Nginx с тегом (версией)
    container_name: my-nginx       # Имя контейнера для удобной идентификации
    restart: unless-stopped        # always: всегда перезапускать, unless-stopped: если не остановлен вручную
    ports:
      - "8080:80"         # 8080 на хосте -> 80 в контейнере (стандартный HTTP порт Nginx)
    volumes:              # Подключение томов (volumes)
      - ./nginx/conf.d:/etc/nginx/conf.d       # Локальная папка   : Путь в контейнере
      - ./html:/usr/share/nginx/html           # Монтирование каталога со статическими файлами
    environment:                               # Переменные окружения (если нужны)
      - NGINX_HOST=localhost
      - NGINX_PORT=80
    working_dir: /usr/share/nginx/html          # Рабочая директория внутри контейнера
    
    # Команды, которые выполнятся при запуске (переопределяют CMD из образа)
    # command: [nginx, '-g', 'daemon off;']
    
    # Зависимости между сервисами (если будут другие сервисы)
    # depends_on:
    #   - app
    #   - database

# Определение сетей (опционально)
# networks:
#   frontend:
#     driver: bridge

# Определение томов (volumes) для постоянного хранения данных
# volumes:
#   nginx_data:
#     driver: local
#   html_content:
#     driver: local
```
