# Admin Key Management Workflow

This document explains the streamlined admin key management system for your Convex self-hosted setup.

## Quick Start

### 1. Start Convex and Generate Admin Key
```bash
pnpm run self-hosted:setup
```
This command will:
- Start the Docker containers
- Wait for the backend to be ready
- Generate and display the admin key
- Save the admin key to a timestamped file in `./admin-key/`

### 2. View Admin Key Files
Admin keys are automatically saved to timestamped markdown files:
```
./admin-key/admin_key_20241220_143022.md
```

## Available Scripts

### Setup & Management
- `pnpm run self-hosted:setup` - Start containers and generate admin key
- `pnpm run self-hosted:dev` - Start containers, deploy functions, and run frontend
- `pnpm run self-hosted:stop` - Stop all containers

### Docker Operations
- `pnpm run docker:up` - Start containers only
- `pnpm run docker:down` - Stop containers
- `pnpm run docker:logs` - View container logs
- `pnpm run docker:generate-admin-key` - Generate admin key (containers must be running)

### Cleanup & Reset
- `pnpm run docker:cleanup-admin-keys` - Remove all saved admin key files
- `pnpm run docker:reset-images` - Stop containers and remove all Docker images/volumes
- `pnpm run docker:reset-full` - Complete cleanup (admin keys + Docker reset)
- `pnpm run self-hosted:reset` - Stop containers and perform full cleanup

## Admin Key File Format

Each generated admin key is saved as a markdown file with:
- Generation timestamp
- Instance name
- The admin key in a code block
- Usage instructions

Example file content:
```markdown
# Convex Admin Key

Generated: Thu Dec 20 14:30:22 PST 2024
Instance Name: convex-self-hosted

## Admin Key
```
your-admin-key-here
```

## Usage
Use this admin key to access the Convex dashboard and manage your self-hosted instance.
```

## Workflow Benefits

1. **Automated**: No manual steps to generate admin keys
2. **Persistent**: Admin keys are saved to files for later reference
3. **Timestamped**: Each key generation creates a new timestamped file
4. **Clean Reset**: Easy cleanup commands for development cycles
5. **Host Accessible**: Admin key files are available on your host machine

## Development Cycle

```bash
# Start fresh
pnpm run self-hosted:setup

# Work with your application...

# Reset everything when needed
pnpm run self-hosted:reset

# Start again
pnpm run self-hosted:setup
```