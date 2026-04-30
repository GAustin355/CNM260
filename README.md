# Firefly III on Docker — CNM260 DevOps Project

A Docker Compose deployment of [Firefly III](https://github.com/firefly-iii/firefly-iii), the open-source self-hosted personal finance manager, paired with a MariaDB backend.

## What is Firefly III?

Firefly III is a free, open-source, self-hosted personal finance manager written in PHP (Laravel). Your transaction data lives in your own MariaDB database — nothing is sent to a third-party service. It supports double-entry bookkeeping across asset, expense, revenue, and liability accounts, plus budgets, categories, tags, bills, and a REST API.

**Upstream project:** https://github.com/firefly-iii/firefly-iii
**License:** [AGPL-3.0](https://github.com/firefly-iii/firefly-iii/blob/main/LICENSE)

## What this repo deploys

Two containers on a private Docker bridge network:

```
+------------------------------------------+
|        Docker network: firefly_iii       |
|                                          |
|   +-----------+      +-----------+       |
|   |    app    | <--> |    db     |       |
|   | Firefly   |      | MariaDB   |       |
|   |   III     |      |   :lts    |       |
|   +-----------+      +-----------+       |
|        |                                 |
|        | port 8080:8080                  |
+--------|---------------------------------+
         v
       Browser -> http://localhost:8080
```

- **app** — `fireflyiii/core:latest` — the Firefly III web application
- **db** — `mariadb:lts` — the database backend
- **firefly_iii_upload** — Docker volume for uploaded receipts/attachments
- **firefly_iii_db** — Docker volume for the database (where your transactions live)

## Quick deploy

```bash
git clone https://github.com/GAustin335/CNM260.git
cd CNM260
cp .env.example .env
cp .db.env.example .db.env
# Edit BOTH files: set APP_KEY (32 chars) and matching DB_PASSWORD / MYSQL_PASSWORD
docker compose up -d --pull=always
```

Open http://localhost:8080 and register the first user.

For the full step-by-step guide assuming no prior Docker experience, see [`deployment-guide.txt`](./deployment-guide.txt).

## Files in this repo

| File | Purpose |
| --- | --- |
| `docker-compose.yml` | The deployment definition — two services, one network, two volumes |
| `.env.example` | Template for the Firefly III app environment (copy to `.env`) |
| `.db.env.example` | Template for the MariaDB environment (copy to `.db.env`) |
| `.gitignore` | Keeps real `.env` / `.db.env` files out of git |
| `LICENSE` | MIT license for the deployment templates |
| `deployment-guide.txt` | Full step-by-step deployment instructions |
| `troubleshooting.txt` | Common errors and fixes |
| `README.md` | This file |

## Companion write-up

Full Medium walkthrough with screenshots: *(link added after publishing)*

## License

Deployment templates: MIT (see [`LICENSE`](./LICENSE))
Firefly III itself: AGPL-3.0, copyright [James Cole](https://github.com/JC5)
