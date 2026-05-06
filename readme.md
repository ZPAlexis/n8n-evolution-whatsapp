### n8n + Evolution API (Docker Starter)

This repository is my personal template for a N8N + Evolution API structure running on Docker for automating the implementation of WhatsApp flows.

### Requirements

- **Docker Desktop**
- **Git**

### Start
Inside the project folder run this command on the terminal:

```bash
docker compose up -d
```

The file `docker-compose.yaml` will start all the required services.

### Access

- **n8n**: `http://localhost:5678`

- **Evolution API**: `http://localhost:8080/manager/`

### Used Commands
```bash
# Status
docker compose ps

# Logs
docker compose logs -f evolution-api

# Restart
docker compose restart

# Stop
docker compose stop

# Stop and remove containers
docker compose down

# Update images & setup containers again
docker compose pull && docker compose up -d --remove-orphans
```

### Stack

- `evolution-api`: API for connecting WhatsApp services.
- `postgres-evolution`: Database.
- `redis-evolution`: Caching.
- `n8n`: Automation platform.

### Others
- **.env**: You will need an `.env` file setup to run this structure. A `.env.example` is provided for example, fill out the missing API_KEYs to run.
- **Used Ports**: Some ports listed might already be used in your machine, edit these ports in `docker-compose.yaml`.
- **Logs**: Use `docker compose logs -f <service>` to check errors in real time.

### Remove all (including volumes)
ATTENTION: This erases all data stored in volumes.

```bash
docker compose down -v
```
### License
Free to use.
