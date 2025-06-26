# Docker Convex Boilerplate

A self-hosted Convex database boilerplate running in Docker with persistence, monitoring, and administration capabilities. Perfect for integrating into monorepos or as a standalone database service.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                Docker Convex Database Boilerplate              │
├─────────────────────────────────────────────────────────────────┤
│  Your Frontend App           │  Convex Functions              │
│  ┌─────────────────────┐     │  ┌─────────────────────────┐   │
│  │ Svelte/React/Vue    │────▶│  │ convex/example.ts       │   │
│  │ - Your UI           │     │  │ - listItems()           │   │
│  │ - Real-time updates │     │  │ - addItem()             │   │
│  │ - Custom features   │     │  │ - deleteItem()          │   │
│  └─────────────────────┘     │  └─────────────────────────┘   │
│           │                  │           │                     │
│           ▼                  │           ▼                     │
│  ┌─────────────────────┐     │  ┌─────────────────────────┐   │
│  │ convex/_generated/  │◀────┼──│ Docker Backend          │   │
│  │ - api.js/api.d.ts   │     │  │ Port 3210 (API)        │   │
│  │ - server.js/.d.ts   │     │  │ Port 3211 (Site Proxy) │   │
│  │ - dataModel.d.ts    │     │  └─────────────────────────┘   │
│  └─────────────────────┘     │           │                     │
│                               │           ▼                     │
│                               │  ┌─────────────────────────┐   │
│                               │  │ Dashboard (Port 6791)   │   │
│                               │  │ - Admin Interface       │   │
│                               │  │ - Database Management   │   │
│                               │  └─────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## Generated Files Explained

The files in `convex/_generated/` are **automatically created** by Convex and are **essential** for your application:

### 🔧 **api.js & api.d.ts**
- **Purpose**: Provide typed API references for your frontend
- **Contains**: `api.example.listItems`, `api.example.addItem`, `api.example.deleteItem` references
- **Used by**: Your frontend app imports these to call your backend functions
- **Auto-regenerated**: Every time you run `convex dev` or `deploy-functions`

### 🔧 **server.js & server.d.ts**
- **Purpose**: Provide server-side utilities (`mutation`, `query`, `action`)
- **Contains**: TypeScript definitions for Convex function builders
- **Used by**: `convex/example.ts` imports `mutation` and `query` from here
- **Auto-regenerated**: When Convex analyzes your schema and functions

### 🔧 **dataModel.d.ts**
- **Purpose**: TypeScript definitions for your database schema
- **Contains**: Table definitions, document types, and ID types
- **Note**: Currently permissive (`Doc = any`) because no schema.ts exists
- **Auto-regenerated**: When you add a `convex/schema.ts` file

**⚠️ DO NOT DELETE OR EDIT THESE FILES** - They're regenerated automatically!

## Files You Can Safely Remove

Some files in this repository are optional or redundant:

### 🗑️ **Files Removed (Boilerplate Cleanup)**
- **`src/`** - ✅ Removed (React frontend - use your own)
- **`index.html`** - ✅ Removed (Vite entry point)
- **`vite.config.mts`** - ✅ Removed (Vite configuration)
- **`convex/chat.ts`** - ✅ Removed (replaced with `example.ts`)
- **Frontend dependencies** - ✅ Removed (React, Vite, etc.)
- **`convex/README.md`** - ✅ Removed (generic documentation)
- **`.idea/`** - ✅ Removed (IDE configuration)
- **Duplicate `.env` files** - ✅ Removed (typos in filenames)

### 📋 **Essential Files**
- **`convex/example.ts`** - Example functions (customize for your needs)
- **`ADMIN_KEY_WORKFLOW.md`** - Admin key management documentation
- **`docker-build/`** - Docker build scripts and utilities
- **`docker-compose.yml`** - Container orchestration
- **All files in `convex/_generated/`** - Auto-generated API bindings

## Customizing Your Functions

Replace the example functions in `convex/example.ts` with your own:

```typescript
// convex/yourFunctions.ts
import { mutation, query } from "./_generated/server";
import { v } from "convex/values";

export const yourQuery = query({
  args: { /* your args */ },
  handler: async (ctx, args) => {
    // Your query logic
  },
});

export const yourMutation = mutation({
  args: { /* your args */ },
  handler: async (ctx, args) => {
    // Your mutation logic
  },
});
```

After adding functions, run `pnpm run deploy-functions` to update the generated API.

## Process Flow

### 1. `self-hosted:setup` Process
```
pnpm run self-hosted:setup
         │
         ▼
┌─────────────────────┐
│ docker:up           │ ──▶ Start containers (backend + dashboard)
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ Wait 10 seconds     │ ──▶ Let backend initialize
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ generate-admin-key  │ ──▶ Create admin credentials
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ Save to admin-key/  │ ──▶ Store timestamped credentials
└─────────────────────┘
```

### 2. `deploy-functions` Process
```
pnpm run deploy-functions
         │
         ▼
┌─────────────────────┐
│ convex dev --once   │ ──▶ Deploy functions to self-hosted instance
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ Analyze convex/     │ ──▶ Scan *.ts files (example.ts, etc.)
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ Generate _generated │ ──▶ Create API bindings & types
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ Push to backend     │ ──▶ Upload functions to Docker instance
└─────────────────────┘
```

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

## Quick Start

Get your Convex database running in one command:

```bash
pnpm run self-hosted:setup
```

This will:
- Start the Docker containers
- Generate admin credentials
- Display the deployment URL, dashboard URL, and admin key
- Save the admin key to a timestamped file in `./admin-key/`

Then deploy your functions:

```bash
pnpm run deploy-functions
```

## Integration with Your Frontend

This boilerplate provides a ready-to-use Convex database. To connect your frontend:

1. **Install Convex in your frontend project:**
   ```bash
   npm install convex
   ```

2. **Set your environment variable:**
   ```bash
   VITE_CONVEX_URL=http://localhost:3210  # or your deployment URL
   ```

3. **Import and use the generated API:**
   ```typescript
   import { api } from "./path/to/convex/_generated/api";
   import { useQuery, useMutation } from "convex/react";
   
   // In your component
   const items = useQuery(api.example.listItems);
   const addItem = useMutation(api.example.addItem);
   ```

## Database Setup

This project runs a Convex database instance in Docker containers, providing a complete database environment with persistence and administration capabilities.

### 1. Configure Database Environment (Optional)

For custom configuration, set up the environment files:

```bash
cp .env.docker.example .env.docker
```

**Configure Database Settings** in `.env.docker`:
- Set a secure `INSTANCE_SECRET` for database encryption
- Configure database ports if needed (defaults: 3210, 3211, 6791)
- Optionally configure storage paths and memory limits

### 2. One-Command Setup (Recommended)

Start everything with a single command:

```bash
pnpm run self-hosted:setup
```

This command will:
- Start Docker containers
- Wait for the backend to be ready
- Generate and display admin credentials
- Save credentials to `./admin-key/admin_key_[timestamp].md`

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

### Quick Setup
- `pnpm run self-hosted:setup` - **One-command setup**: Start containers and generate admin key
- `pnpm run deploy-functions` - Deploy schema changes

### Core Database Operations
- `pnpm run docker:up` - Start the database server
- `pnpm run docker:down` - Stop the database server
- `pnpm run docker:logs` - View database logs

### Administration
- `pnpm run docker:generate-admin-key` - Generate new admin credentials
- `pnpm run self-hosted:setup-manual` - Manual database initialization
- `pnpm run self-hosted:reset` - Stop services and perform full Docker reset

### Maintenance
- `pnpm run docker:cleanup-admin-keys` - Remove saved admin key files
- `pnpm run docker:reset-images` - Stop Docker and prune system volumes
- `pnpm run docker:reset-full` - Combine key cleanup and image reset

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

1. **Quick Setup (Recommended):**
   ```bash
   pnpm run self-hosted:setup
   ```
   This will display the deployment URL, dashboard URL, and admin key.

2. **Or generate credentials separately:**
   ```bash
   pnpm run docker:generate-admin-key
   ```

3. **Use the generated key for dashboard login** - Format:
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
