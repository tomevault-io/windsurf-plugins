---
trigger: always_on
description: Makefile and shell scripting patterns for POS System development workflow
---


# Makefile & Shell Scripting Guidelines

## Makefile Structure

### Main Makefile Organization
Follow the patterns established in [Makefile](mdc:Makefile):

### Standard Makefile Headers
```makefile
# POS System - Development Makefile
# Usage: make <command>

.PHONY: help dev prod up down build logs clean backup restore create-admin remove-data db-shell test lint format

# Default target
.DEFAULT_GOAL := help

# Colors for output
GREEN := \033[0;32m
YELLOW := \033[0;33m
RED := \033[0;31m
BLUE := \033[0;34m
NC := \033[0m # No Color
```

### Command Categories
Organize commands into logical groups:
1. **Development Commands** - dev, prod, up, down, restart, build
2. **Database Commands** - create-admin, remove-data, backup, restore, db-shell, db-reset
3. **Utility Commands** - logs, status, clean, test, lint, format
4. **Quick Shortcuts** - start, stop, install

### Help System Pattern
```makefile
## Help - Display available commands
help:
	@echo "$(BLUE)POS System - Available Make Commands$(NC)"
	@echo ""
	@echo "$(GREEN)Development Commands:$(NC)"
	@echo "  make dev          - Start development environment with hot reloading"
	# ... more help text
```

## Docker Integration

### Docker Compose File Selection
```makefile
# Docker compose files
COMPOSE_DEV := docker-compose.dev.yml
COMPOSE_PROD := docker-compose.yml

# Development environment
dev:
	@docker compose -f $(COMPOSE_DEV) up --build

# Production environment  
prod:
	@docker compose -f $(COMPOSE_PROD) up -d --build
```

### Container Management
```makefile
# Check container status before operations
up:
	@echo "$(GREEN)⬆️ Starting Docker containers...$(NC)"
	@docker compose -f $(COMPOSE_DEV) up -d
	@echo "$(GREEN)✅ Containers started in background$(NC)"

down:
	@echo "$(YELLOW)⬇️ Stopping Docker containers...$(NC)"
	@docker compose -f $(COMPOSE_DEV) down
	@docker compose -f $(COMPOSE_PROD) down 2>/dev/null || true
	@echo "$(GREEN)✅ Containers stopped$(NC)"
```

## Shell Script Patterns

### Script Structure
Follow the patterns from [scripts/](mdc:scripts/) directory:

### Standard Script Headers
```bash
#!/bin/bash

# Script Description
# Brief description of what this script does

set -e  # Exit on any error

# Colors
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
RED='\033[0;31m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color
```

### Container Detection Pattern
```bash
# Check if database container is running
CONTAINER_NAME="pos-postgres-dev"
if ! docker ps --format '{{.Names}}' | grep -q "^${CONTAINER_NAME}$"; then
    CONTAINER_NAME="pos-postgres"
    if ! docker ps --format '{{.Names}}' | grep -q "^${CONTAINER_NAME}$"; then
        echo -e "${RED}❌ Database container is not running!${NC}"
        echo -e "${YELLOW}Please run 'make up' or 'make dev' first.${NC}"
        exit 1
    fi
fi
```

### User Input Validation
```bash
# Safe user input with validation
read -p "Username: " USERNAME
while [[ -z "$USERNAME" ]]; do
    echo -e "${RED}Username cannot be empty!${NC}"
    read -p "Username: " USERNAME
done

# Hidden password input
echo -n "Password: "
read -s PASSWORD
echo ""
```

### Confirmation Patterns
```bash
# Multiple confirmation levels for destructive operations
echo -e "${RED}⚠️  WARNING: This will DELETE ALL DATA!${NC}"
echo -e "${YELLOW}Type 'DELETE ALL DATA' to confirm (case sensitive):${NC}"
read -p "> " FINAL_CONFIRMATION

if [[ "$FINAL_CONFIRMATION" != "DELETE ALL DATA" ]]; then
    echo -e "${BLUE}❌ Operation cancelled. Data is safe.${NC}"
    exit 0
fi
```

### Database Operations
```bash
# Database backup with error handling
echo -e "${YELLOW}💾 Creating backup...${NC}"
DB_BACKUP_FILE="${BACKUP_DIR}/${BACKUP_NAME}_database.sql"

docker exec $CONTAINER_NAME pg_dump \
    -U postgres \
    -d pos_system \
    --clean \
    --if-exists \
    --create \
    --verbose \
    --format=plain > $DB_BACKUP_FILE

if [[ $? -eq 0 ]]; then
    DB_SIZE=$(du -h $DB_BACKUP_FILE | cut -f1)
    echo -e "${GREEN}✅ Database backup created: $DB_BACKUP_FILE ($DB_SIZE)${NC}"
else
    echo -e "${RED}❌ Database backup failed!${NC}"
    exit 1
fi
```

## Error Handling

### Comprehensive Error Checking
```bash
# Always check command success
if [[ $? -eq 0 ]]; then
    echo -e "${GREEN}✅ Operation completed successfully!${NC}"
else
    echo -e "${RED}❌ Operation failed!${NC}"
    exit 1
fi
```

### Emergency Backup Pattern
```bash
# Create emergency backup before destructive operations
EMERGENCY_BACKUP="$BACKUP_DIR/emergency_backup_$(date +%Y%m%d_%H%M%S).sql"
echo -e "${BLUE}Creating emergency backup: $EMERGENCY_BACKUP${NC}"
docker exec $CONTAINER_NAME pg_dump -U postgres pos_system > $EMERGENCY_BACKUP
```

## File and Directory Management

### Backup Directory Structure
```bash
# Create backup directory if it doesn't exist
BACKUP_DIR="backups"
mkdir -p $BACKUP_DIR

# Generate timestamp for backup files
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
BACKUP_NAME="pos_backup_${TIMESTAMP}"
```

### File Size and Statistics
```bash
# Show file sizes and statistics
if [[ -f "$DB_BACKUP_FILE" ]]; then
    DB_SIZE=$(du -h $DB_BACKUP_FILE | cut -f1)
    echo -e "${GREEN}✅ Database backup: $DB_BACKUP_FILE ($DB_SIZE)${NC}"
fi

# Count records in database
USER_COUNT=$(docker exec $CONTAINER_NAME psql -U postgres -d pos_system -tAc "SELECT COUNT(*) FROM users;")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/poinf-of-sales](https://github.com/madebyaris/poinf-of-sales) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
