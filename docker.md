- This is a bind mount.

- It mounts a single file from your host (./init.sql) into the container at startup.

- Purpose → Run your custom SQL script the first time the container is initialized.

-  But it does not persist the database data itself.

- If you remove the container, all DB data is lost (unless you export/dump it).
```
volumes:
  - ./init.sql:/docker-entrypoint-initdb.d/init.sql

```

### 2. Persistent volume setup

- This uses a named Docker volume (postgres_data).

- /var/lib/postgresql/data is the folder inside the container where Postgres stores all DB files (tables, indexes, WAL logs, etc.).

- Docker will create a persistent volume on your system (managed by Docker, not tied to your local folder).

- ✅ Benefit → Even if you delete/recreate the container, your database data persists.

```
volumes:
  - postgres_data:/var/lib/postgresql/data

```

### How Docker Volumes Work

- A named volume (like postgres_data) is managed by Docker and stored under:

```

/var/lib/docker/volumes/<volume_name>/_data

```
- If you reuse the same volume name across projects, they will share the same database files.

- If you want project-specific data, you need unique volume names per project.

### How Docker Handles It

- Inside every Postgres container, Postgres expects to write data to /var/lib/postgresql/data.

- When you attach a volume, Docker decides where on your host machine that data actually lives.
- example:
```
volumes:
  - raga_postgres_data:/var/lib/postgresql/data

```

- Inside container → /var/lib/postgresql/data

- On host → /var/lib/docker/volumes/raga_postgres_data/_data

#### If another project uses:

```
volumes:
  - myapp_postgres_data:/var/lib/postgresql/data

```

- Inside container → still /var/lib/postgresql/data

- On host → /var/lib/docker/volumes/myapp_postgres_data/_data

##### Note:- Even though the container path is the same, the host path is different because the volume names differ. That’s how Docker keeps project data isolated.

- This shows all volumes (raga_postgres_data, myapp_postgres_data, etc).
```
docker volume ls

```
- run to know mount point
```
docker volume inspect raga_postgres_data
```

- Output will show the Mountpoint, e.g.:

```
"Mountpoint": "/var/lib/docker/volumes/raga_postgres_data/_data"

```

- example of docker compose file

```yml

services:
  postgres:
    image: postgres
    container_name: raga-postgres
    environment:
      POSTGRES_DB: raga
      POSTGRES_USER: raga_user
      POSTGRES_PASSWORD: raga_password
    ports:
      - "5432:5432"
    volumes:
      - raga_postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql

volumes:
  raga_postgres_data:


```