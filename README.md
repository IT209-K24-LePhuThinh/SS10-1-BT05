# BT05 - Tự động hóa điều phối đa dịch vụ với Docker Compose & File .env

## 1. docker-compose.yml

```yaml
services:
  database:
    image: postgres:15-alpine
    container_name: db-server
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - backend-net

  backend:
    build: .
    container_name: app-container
    ports:
      - "${PORT}:3000"
    environment:
      DB_HOST: database
      DB_PORT: ${DB_PORT}
      DB_NAME: ${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
    depends_on:
      - database
    networks:
      - backend-net

volumes:
  pgdata:

networks:
  backend-net:
```

## 2. .env.example

```dotenv
DB_NAME=appdb
DB_USER=appuser
DB_PASSWORD=change_me
DB_PORT=5432
PORT=8080
```

Tệp `.env` thật chứa cùng các biến trên với mật khẩu riêng, không đưa lên Git.

## 3. .gitignore

```gitignore
.env
node_modules/
```

## 4. Khởi chạy toàn bộ hệ thống

```bash
devops@0310263186:~/hw/session_10/ex5$ docker compose up -d
[+] Running 5/5
 ✔ Network ex5_backend-net   Created
 ✔ Volume "ex5_pgdata"       Created
 ✔ Container db-server       Started
 ✔ Container app-container   Started
```

```bash
devops@0310263186:~/hw/session_10/ex5$ docker compose ps
NAME            IMAGE               COMMAND                  SERVICE    CREATED          STATUS          PORTS
app-container   ex5-backend         "docker-entrypoint.s…"   backend    8 seconds ago    Up 7 seconds    0.0.0.0:8080->3000/tcp
db-server       postgres:15-alpine  "docker-entrypoint.s…"   database   8 seconds ago    Up 7 seconds    5432/tcp
```

```bash
devops@0310263186:~/hw/session_10/ex5$ docker compose logs -f
db-server  | PostgreSQL init process complete; ready for start up.
db-server  | LOG:  database system is ready to accept connections
app-container  | Server listening on port 3000
```

Kiểm tra API backend sau khi Compose tự tạo network và volume:

```bash
devops@0310263186:~/hw/session_10/ex5$ curl -i http://localhost:8080/
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 71
Date: Mon, 12 Oct 2026 15:47:33 GMT
Connection: keep-alive
Keep-Alive: timeout=5

{"status":"success","message":"Hello from Docker Single-stage!"}
```

## 5. Dọn dẹp

```bash
devops@0310263186:~/hw/session_10/ex5$ docker compose down
[+] Running 3/3
 ✔ Container app-container   Removed
 ✔ Container db-server       Removed
 ✔ Network ex5_backend-net   Removed
```
