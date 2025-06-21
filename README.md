# Dockerized Convex Database

A self-contained Convex database instance running in Docker with persistence, monitoring, and administration capabilities. Ideal for local development, testing, and standalone database deployments.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Setup Guide](#setup-guide)
  - [Installation](#1-clone-repository)
  - [Database Initialization](#2-initialize-database-automated)
  - [Database Operations](#3-database-operations)
  - [Manual Configuration](#4-manual-database-setup)
- [Database Management Commands](#database-management-commands)
- [Database Access](#database-access)
- [Troubleshooting](#troubleshooting)

## Prerequisites

- [Docker](https://www.docker.com/get-started/) for database containerization
- [Node.js](https://nodejs.org/) (v18+) for running management tools
- [pnpm](https://pnpm.io/) (v8+) for package management

---

## Database Setup

This project runs a Convex database instance in Docker containers, providing a complete database environment with persistence and administration capabilities.

### 1. Configure Database Environment

- Set up the environment configuration:
  ```bash
  cp .env.docker.example .devcontainer/.env.docker
  cp .env.local.example .devcontainer/.env.local
  ```

- **Configure Database Settings** in `.devcontainer/.env.docker`:
  - Set a secure `INSTANCE_SECRET` for database encryption
  - Configure database ports if needed (defaults: 3210, 3211, 6791)
  - Optionally configure storage paths and memory limits

- **Configure Access Settings** in `.devcontainer/.env.local`:
  ```bash
  # Database connection URL
  CONVEX_SELF_HOSTED_URL=http://localhost:3210
  
  # Admin key will be auto-generated on first startup
  # CONVEX_SELF_HOSTED_ADMIN_KEY=
  
  # Ensure CONVEX_DEPLOYMENT is commented out for local database
  # CONVEX_DEPLOYMENT=
  ```

### 2. Initialize Database (Automated)

Run the automated setup script to initialize your database environment:

```bash
./post-create-setup.sh
```

This script will:

- Configure environment settings
- Initialize Docker containers
- Generate database admin credentials
- Set up initial database schema
- Configure data persistence

### 3. Database Operations

Start the database server:

```bash
pnpm run docker:up
```

Stop the database server:

```bash
pnpm run docker:down
```

Deploy schema changes:

```bash
pnpm run deploy-functions
```

### 4. Manual Database Setup

For manual control over the database setup:

1. Start database and create admin credentials:
   ```bash
   pnpm run self-hosted:setup-manual
   ```

2. Configure admin access in `.devcontainer/.env.local`:
   ```bash
   CONVEX_SELF_HOSTED_ADMIN_KEY=<generated-admin-key>
   ```

3. Initialize database schema:
   ```bash
   pnpm run deploy-functions
   ```

---

---

## Database Management Commands

### Core Database Operations
- `pnpm run docker:up` - Start the database server
- `pnpm run docker:down` - Stop the database server
- `pnpm run docker:logs` - View database logs
- `pnpm run deploy-functions` - Deploy schema changes

### Administration
- `pnpm run docker:generate-admin-key` - Generate new admin credentials
- `pnpm run self-hosted:setup-manual` - Manual database initialization

### Maintenance
- `pnpm run post-create` - Initialize database environment
- `pnpm run post-start` - Perform startup checks and maintenance

> All commands use `pnpm`. Ensure it's installed via `npm install -g pnpm` if needed.

---

## Security Considerations

### Access Control
- Keep admin credentials secure and rotate them regularly
- Use environment variables for sensitive configuration
- Never commit `.env` files containing credentials
- Restrict database ports to localhost when possible

### Network Security
- Configure firewall rules to restrict database access
- Use TLS/SSL for external connections
- Monitor access logs for suspicious activity
- Keep Docker and database software updated

### Data Protection
- Implement regular backup procedures
- Encrypt sensitive data at rest
- Follow the principle of least privilege
- Document all security configurations

---

## Database Access

### Connection Endpoints

- **Database API**: [http://localhost:3210](http://localhost:3210)
- **Admin Dashboard**: [http://localhost:6791](http://localhost:6791)

### Database Administration

The Convex Admin Dashboard at [http://localhost:6791](http://localhost:6791) provides a web interface for:
- Monitoring database health
- Managing data and schemas
- Viewing logs and metrics
- Running queries

### Admin Authentication

To access the admin dashboard:

1. **Generate admin credentials:**
   ```bash
   pnpm run docker:generate-admin-key
   ```

2. **Use the generated key for dashboard login** - Format:
   ```
   instance-name|admin-key-hash
   ```
   Example: `convex-tutorial-local|01f7a735340227d2769630a37069656211287635ea7cc4737e98d517cbda803723deccea7b4b848d1d797975102c2c7328`

3. **Credential Storage**: The admin key is automatically saved in `.devcontainer/.env.local` as `CONVEX_SELF_HOSTED_ADMIN_KEY`

> **Security Note**: The admin key grants full database access. Store it securely and never share it.

---

## Troubleshooting

### Database Connection Issues

If unable to connect to the database:
1. Verify Docker container status: `docker ps`
2. Check database logs: `pnpm run docker:logs`
3. Ensure ports 3210 and 6791 are available
4. Validate connection settings in `.devcontainer/.env.local`

### Admin Dashboard Access

If dashboard authentication fails:
1. Generate new credentials: `pnpm run docker:generate-admin-key`
2. Verify admin key in `.devcontainer/.env.local`
3. Use complete key format: `instance-name|admin-key-hash`

### Schema Deployment Issues

If schema changes aren't reflecting:
1. Execute `pnpm run deploy-functions`
2. Check deployment logs for errors
3. Verify schema file syntax

### Data Persistence

If data isn't persisting between restarts:
1. Check Docker volume configuration
2. Verify database shutdown was clean
3. Review backup settings if configured

### Database Management

```bash
# Monitor database status
docker ps --filter "name=convex"

# View detailed database logs
docker compose logs convex-backend

# Inspect database volumes
docker volume ls --filter "name=convex"
```

### Data Backup and Recovery

```bash
# Create full database backup
docker run --rm --volumes-from convex-backend -v $(pwd):/backup alpine tar cvf /backup/convex-data.tar /data

# Export database schema
pnpm run deploy-functions -- --dry-run > schema-backup.json

# Restore from backup
docker run --rm --volumes-from convex-backend -v $(pwd):/backup alpine tar xvf /backup/convex-data.tar

# Reset database (⚠️ Warning: Deletes all data!)
docker compose down -v && docker volume prune -f
```

### Maintenance

```bash
# Rebuild database container
docker compose build convex-backend

# Check database health
curl http://localhost:3210/_system/health

# View database metrics
curl http://localhost:3210/_system/metrics
```

---

## Security Notes

- **Always change the `INSTANCE_SECRET` in `.env.docker` before deploying to any non-local environment!**
- **Keep your admin key secure - it provides full access to your Convex backend**
- **Never commit environment files containing secrets to version control**

---

#### Port Configuration

The following ports are automatically forwarded:

- **5173**: Vite development server (your React app)
- **3210**: Convex backend server
- **6791**: Convex dashboard

#### Accessing Your App

Once the development server is running:

1. Click on the "Ports" tab in VS Code
2. Click the globe icon next to port 5173 to open your app
3. Access the Convex dashboard on port 6791 (use the admin key as password)
