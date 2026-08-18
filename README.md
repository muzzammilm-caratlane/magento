# Magento Docker Environment

Local Docker-based development environment for Magento, using the `bin/*` helper scripts to manage containers, imports, and configuration.

## Prerequisites

- Docker installed
- SSH access configured for GitHub (`git@github.com`)
- Access to your existing Magento source repository and database dump

## Getting Started

### 1. Create your project directory and enter it

```bash
mkdir -p ~/Sites/magento
cd $_
```

### 2. Download the Docker Compose template

```bash
git clone git@github.com:muzzammilm-caratlane/magento.git .
```

### 3. Back up your existing database

```bash
bin/mysqldump > ~/Sites/existing/magento.sql
```

### 4. Add your existing Magento source code

Replace the `src` directory with your existing Magento instance's source code:

```bash
git clone git@github.com:myrepo.git src
```

### 5. Start containers and copy files

```bash
bin/start --no-dev
bin/copytocontainer --all   # Initial copy will take a few minutes...
```

### 6. Install Composer dependencies (if `vendor` is empty)

```bash
bin/composer install
```

### 7. Import your existing database

```bash
bin/mysql < ../existing/magento.sql
```

### 8. Import app-specific environment settings

```bash
bin/magento app:config:import
```

### 9. Set up DNS and base URL

```bash
bin/setup-domain upgrade.caratlane.us
bin/restart
open upgrade.caratlane.us
```

## Multi-Store Setup

1. Set up `nginx.conf` as configured in `src`.
2. Run the following:

```bash
bin/setup upgrade.caratlane.us upgradestore.caratlane.us
bin/setup-ssl upgrade.caratlane.us upgradestore.caratlane.us
bin/setup-ssl-ca upgrade.caratlane.us upgradestore.caratlane.us
```

## Elasticsearch Indexing

Enable ID field data on the cluster:

```bash
curl -X PUT "localhost:9200/_cluster/settings" -H "Content-Type: application/json" -d'
{
  "persistent": {
    "indices.id_field_data.enabled": true
  }
}'
```

Reindex and clear caches:

```bash
bin/magento indexer:reindex
bin/magento cache:clean
bin/magento cache:flush
```

## Static Content Signing

Also configure static content signing per Adobe's documentation:
https://experienceleague.adobe.com/en/docs/commerce-operations/configuration-guide/cache/static-content-signing

## Working with the Environment

`src/app` is bind-mounted to the container:

- **Most JS changes** reflect automatically. If not, remove the compiled static frontend files.
- **Frontend changes** generally just require purging the corresponding cache from the Admin panel.
- **Anything updated outside of `/app`** must be manually copied into the container using `bin/copytocontainer`.

## Command Reference

| Command | Description |
|---|---|
| `bin/copyfromcontainer` | Copy folders or files from container to host. Ex: `bin/copyfromcontainer vendor` |
| `bin/copytocontainer` | Copy folders or files from host to container. Ex: `bin/copytocontainer --all` |
| `bin/create-user` | Create either an admin user or customer account |
| `bin/cron` | Start or stop the cron service. Ex: `bin/cron start` |
| `bin/fixowns` | Fix filesystem ownerships within the container |
| `bin/fixperms` | Fix filesystem permissions within the container |
| `bin/redis` | Run a command from the redis container. Ex: `bin/redis redis-cli monitor` |
| `bin/remove` | Remove all containers |
| `bin/removeall` | Remove all containers, networks, volumes, and images (calls `bin/stopall` first) |
| `bin/removenetwork` | Remove the network associated with the current directory's name |
| `bin/removevolumes` | Remove all volumes |
| `bin/restart` | Stop and then start all containers |
| `bin/start` | Start all containers (preferred over `docker-compose up -d`, may include extra helpers) |
| `bin/status` | Check container status |
| `bin/stop` | Stop all project containers |
| `bin/stopall` | Stop all running Docker containers on the host |
