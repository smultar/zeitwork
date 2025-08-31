# Docker Compose Setup for Zeitwork

This docker-compose configuration sets up a complete development environment for the Zeitwork platform.

## Quick Start

1. **Clone the repository** (if you haven't already):
   ```bash
   git clone https://github.com/smultar/zeitwork.git
   cd zeitwork
   ```

2. **Copy the environment file and configure it**:
   ```bash
   cp .env.example .env
   ```
   
   Edit the `.env` file with your GitHub OAuth and App credentials:
   - `GITHUB_CLIENT_ID` and `GITHUB_CLIENT_SECRET`: Create a GitHub OAuth App
   - `GITHUB_APP_ID` and `GITHUB_APP_PRIVATE_KEY`: Create a GitHub App for repository integration

3. **Start the services**:
   ```bash
   docker-compose up -d
   ```
   
   **⏱️ First Startup Note**: The initial startup may take 3-5 minutes as the web service needs to download and install all npm dependencies. Subsequent startups will be much faster due to volume caching.

4. **Monitor the startup progress**:
   ```bash
   # Watch the logs to see startup progress
   docker-compose logs -f web
   
   # Check when services are ready
   docker-compose ps
   ```

5. **Access the application**:
   - Web interface: http://localhost:3000
   - Backend metrics: http://localhost:8080 (if running)
   - Database: localhost:5432

## Services

### PostgreSQL Database
- **Port**: 5432
- **Database**: zeitwork
- **User**: postgres
- **Password**: postgres
- **Includes**: Automatic schema migration on startup

### Web Application (Nuxt.js)
- **Port**: 3000
- **Technology**: Nuxt.js with TypeScript, Vue 3, TailwindCSS
- **Features**: 
  - GitHub OAuth authentication
  - Project management interface
  - REST API endpoints
  - Real-time development with hot reload

### Backend (Kubernetes Controller)
- **Ports**: 8080 (metrics), 8081 (health)
- **Technology**: Go with controller-runtime
- **Note**: This is designed for Kubernetes environments and may show errors in docker-compose, but provides metrics endpoint

## Development Workflow

### Working with the Web Application
```bash
# View logs
docker-compose logs -f web

# Execute commands in the web container
docker-compose exec web bun run dev

# Access the container shell
docker-compose exec web sh
```

### Working with the Database
```bash
# Connect to PostgreSQL
docker-compose exec postgres psql -U postgres -d zeitwork

# View database logs
docker-compose logs postgres
```

### Rebuilding Services
```bash
# Rebuild and restart a specific service
docker-compose build web && docker-compose up -d web

# Rebuild all services
docker-compose build && docker-compose up -d
```

## Configuration

### Environment Variables

The following environment variables can be configured in your `.env` file:

| Variable | Description | Default |
|----------|-------------|---------|
| `GITHUB_CLIENT_ID` | GitHub OAuth App Client ID | Required |
| `GITHUB_CLIENT_SECRET` | GitHub OAuth App Client Secret | Required |
| `GITHUB_APP_ID` | GitHub App ID for repository access | Required |
| `GITHUB_APP_PRIVATE_KEY` | GitHub App Private Key | Required |
| `SESSION_PASSWORD` | Session encryption password (32+ chars) | Required |
| `APP_URL` | Application base URL | http://localhost:3000 |

### GitHub Setup

1. **Create a GitHub OAuth App**:
   - Go to GitHub Settings > Developer settings > OAuth Apps
   - Create a new OAuth App with:
     - Application name: "Zeitwork Local"
     - Homepage URL: http://localhost:3000
     - Authorization callback URL: http://localhost:3000/auth/github

2. **Create a GitHub App**:
   - Go to GitHub Settings > Developer settings > GitHub Apps
   - Create a new GitHub App with:
     - GitHub App name: "Zeitwork Local"
     - Homepage URL: http://localhost:3000
     - Webhook URL: http://localhost:3000/api/github/webhook
     - Permissions: Repository (Read & Write), Metadata (Read), Pull requests (Read)

## Troubleshooting

### Common Issues

1. **Slow initial startup**:
   The first startup can take 3-5 minutes as npm installs all dependencies for the web application. This is normal and subsequent startups will be much faster.
   ```bash
   # Monitor installation progress
   docker-compose logs -f web
   ```

2. **Database connection errors**:
   ```bash
   # Check if PostgreSQL is healthy
   docker-compose ps postgres
   
   # Restart PostgreSQL
   docker-compose restart postgres
   ```

3. **Web application won't start**:
   ```bash
   # Check the logs
   docker-compose logs web
   
   # Ensure environment variables are set
   cat .env
   ```

4. **Port conflicts**:
   ```bash
   # Check what's using the ports
   lsof -i :3000
   lsof -i :5432
   
   # Change ports in docker-compose.yaml if needed
   ```

5. **Backend errors** (expected):
   The backend is a Kubernetes controller and will show errors about missing Kubernetes cluster. This is normal for local development.

### Resetting Everything

```bash
# Stop all services and remove volumes
docker-compose down -v

# Remove all images
docker-compose down --rmi all

# Start fresh
docker-compose up -d
```

## Architecture

- **apps/web/**: Nuxt.js full-stack web application with integrated API
- **apps/backend/**: Go-based Kubernetes controller (for production Kubernetes deployments)
- **apps/k8s/**: Pulumi infrastructure as code (for production deployments)

The web application includes its own API server, so you can develop the frontend and basic functionality without needing the Kubernetes backend running.