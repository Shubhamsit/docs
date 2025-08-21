# 📘 pgAdmin Setup Guide for Multiple Projects

This guide explains how to run **pgAdmin in Docker** as a standalone service and connect it to different PostgreSQL databases across multiple projects.

---

## 🚀 1. Setup pgAdmin with Docker

Create a folder (e.g., `~/pgadmin/`) and add a `docker-compose.yml`:

```yaml
version: '3.8'

services:
  pgadmin:
    image: dpage/pgadmin4
    container_name: pgadmin
    restart: always
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - "5050:80"
    volumes:
      - pgadmin_data:/var/lib/pgadmin

volumes:
  pgadmin_data:
```

Start pgAdmin:
```bash
docker-compose up -d
```

👉 Access it at: [http://localhost:5050](http://localhost:5050)  
👉 Login: `admin@admin.com / admin`  

The volume `pgadmin_data` ensures your **saved connections persist** even if the container restarts.

---

## 📂 2. Connecting pgAdmin to Different Projects

Each project may have its own PostgreSQL database. You can connect all of them inside pgAdmin.

### Example: Project A (Dockerized PostgreSQL)
- **Host**: `localhost` (if PostgreSQL is exposed with `ports: "5432:5432"`)  
- **Port**: `5432`  
- **Username**: the `POSTGRES_USER` you set in `docker-compose.yml`  
- **Password**: the `POSTGRES_PASSWORD` you set  

Inside pgAdmin:
1. Right-click **Servers > Register > Server**  
2. Enter:
   - **Name**: `Project A DB`
   - **Host**: `localhost`
   - **Port**: `5432`
   - **Username**: (your user)
   - **Password**: (your password)

---

### Example: Project B (Different port / credentials)
If Project B runs on port `5433` with different credentials:  
- **Host**: `localhost`  
- **Port**: `5433`  
- **Username**: `projectb_user`  
- **Password**: `projectb_pass`  

Add it as another server in pgAdmin:
- **Name**: `Project B DB`

---

### Example: Remote/Production Database
If you have a production database on a remote server:
- **Host**: `prod.db.example.com`
- **Port**: `5432`
- **Username**: `prod_user`
- **Password**: `securepassword`

Add it as another server in pgAdmin:
- **Name**: `Production DB`

---

## 🗂️ 3. Final pgAdmin View

After connecting multiple projects, pgAdmin will list them separately:

```
Servers
 ├── Project A DB
 ├── Project B DB
 └── Production DB
```

You can now manage **all projects’ databases in one place**.

---

## 🔑 Notes

- If using **Docker Compose**, make sure Postgres service exposes the port:
  ```yaml
  ports:
    - "5432:5432"
  ```

- If you don’t want to expose Postgres directly, you can run pgAdmin **inside the same Docker network** and connect via container name instead of `localhost`.  

Example:
```yaml
host: db  # service name in docker-compose
```

---

✅ With this setup, pgAdmin acts as a **central DB manager** for all your projects.