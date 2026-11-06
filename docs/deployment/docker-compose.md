# Docker Compose Deployment

The simplest way to run KubeSight locally or in development.

## Quick Start

```bash
git clone https://github.com/kubesight-ai/kubesight.git
cd kubesight
docker-compose up -d
```

Access at **http://localhost:3000**

## docker-compose.yml

```yaml
services:
  postgres:
    image: postgres:16-alpine
    container_name: kubesight-postgres
    environment:
      POSTGRES_DB: kubesight
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    container_name: kubesight-redis

  backend:
    build: ./backend
    container_name: kubesight-backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql+asyncpg://postgres:postgres@postgres:5432/kubesight
      - K8S_CONFIG_PATH=/home/kubesight/.kube/config
      - GEMINI_API_KEY=${GEMINI_API_KEY}
    volumes:
      - ~/.kube:/home/kubesight/.kube:ro
    depends_on:
      - postgres
      - redis

  frontend:
    build: ./frontend
    container_name: kubesight-frontend
    ports:
      - "3000:80"
    depends_on:
      - backend
```

## Environment Configuration

Create a `.env` file:

```env
# AI Provider
GEMINI_API_KEY=your-api-key

# Optional
DEBUG=true
```

## Common Commands

```bash
# Start services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down

# Rebuild after changes
docker-compose up -d --build

# Remove volumes
docker-compose down -v
```

## Using Makefile

```bash
make dev     # Start development
make logs    # View logs
make down    # Stop services
make clean   # Remove containers and volumes
```

## Troubleshooting

### Port Already in Use

```bash
# Check what's using the port
lsof -i :3000
lsof -i :8000

# Use different ports (edit docker-compose.yml)
# Change ports mapping in frontend service
```

### Kubernetes Connection Issues

Ensure your kubeconfig is accessible:

```bash
# Test kubectl access
docker-compose exec kubesight-backend kubectl get nodes
```

### Logs Not Streaming

Check WebSocket connectivity:

```bash
# Test WebSocket
wscat -c ws://localhost:8000/api/v1/ws/health
```
