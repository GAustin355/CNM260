Companion write-up

Full Medium walkthrough with screenshots: https://medium.com/@gaustin_63873/self-hosting-your-money-deploying-firefly-iii-on-docker-f81c249ed8a8?postPublishedType=repub

Self-Hosting Your Money: Deploying Firefly III on Docker
Two containers, one config file pair, and every command you need to copy & paste.

Why Bother Self Hosting Your Finances?
Mint shut down. YNAB costs over $100 a year. Quicken still wants you to
install software from a CD. Meanwhile, every cloud finance app means your financial data sits on someone else’s server.

Firefly III is a free, open-source, self-hosted finance manager. It’s a
PHP/Laravel application that runs in Docker, stores your transactions in
your own MariaDB database, and never phones home. You get budgets,
categories, tags, accounts, recurring transactions, bill reminders, and an
OAuth API.

This guide walks through deploying it using Docker Compose, with a MariaDB backend, in about fifteen minutes. Everything you need is in this companion repo: https://github.com/GAustin355/CNM260

What we are Building?
Two containers running on a private Docker network:
   
    +------------------------------------------+
    |        Docker network: firefly_iii       |
    |                                          |
    |   +-----------+      +-----------+       |
    |   |    app    | <--> |    db     |       |
    |   |  Firefly  |      |  MariaDB  |       |
    |   |    III    |      |   :lts    |       |
    |   +-----------+      +-----------+       |
    |        |                                 |
    |        |    port 8080:8080               |
    +--------|---------------------------------+
             v
           Browser -> http://localhost:8080

The “app” container runs Firefly III. The “db” container runs MariaDB.

They talk to each other inside Docker; only the app exposes a port to your
browser.

What You’ll Need Before You Start
Computer (Windows, Mac, or Linux)
Docker Desktop — runs the containers
A terminal (PowerShell on Windows, Terminal on Mac, etc.)
~500 MB free disk space
Any text editor — Notepad works just fine for this.
Step 1 — Install Docker

WINDOWS / macOS:

Go to https://www.docker.com/products/docker-desktop and download Docker
Desktop. Run the installer. On Windows, accept the WSL 2 prompts and
restart. Launch Docker Desktop and wait for the whale icon to stop animating.

LINUX (Ubuntu / Debian):

sudo apt update
sudo apt install -y docker.io docker-compose-plugin
sudo usermod -aG docker $USER
Log out and back in so the group change takes effect.

Verify:

docker — version
docker compose version
Both commands should print version numbers.

Step 2 — Get the deployment files

Open a terminal, navigate to wherever you want the project to live, and
clone the repo:

git clone https://github.com/GAustin355/CNM260.git

cd CNM260
Don’t have git? On the GitHub page, click the green Code button -> Download ZIP, unzip it, and “cd” into the unzipped folder.

Step 3 — Copy the example config files
The repo ships your .env.example and .db.env.example files. You’re going to copy them, drop the .example suffix, and fill in your own secrets.

cp .env.example .env
cp .db.env.example .db.env
(On Windows PowerShell, use Copy-Item instead of cp.)

Step 4 — Generate a 32-character APP_KEY
Firefly III needs a 32-character random string to encrypt session data.
Don’t pick one yourself — humans pick bad random strings. Use this command:

macOS / Linux / WSL:

head /dev/urandom | LC_ALL=C tr -dc ‘A-Za-z0–9’ | head -c 32 && echo
Windows PowerShell:

-join ((48..57) + (65..90) + (97..122) | Get-Random -Count 32 | ForEach-Object {[char]$_})
You’ll get something like:

aB3kP9xQ2nR7vL1mT4jH6dF8wG5sZ0yC

Copy it. You’ll paste it next.

IMPORTANT:

Don’t put any of these characters in the APP_KEY: # < > = &
They break Laravel’s parser. The generators above won’t produce them, so
you’re safe as long as you use the generators.

Step 5 — Edit .env and .db.env
Open .env in a text editor.

Find: APP_KEY=CHANGE_ME_TO_A_RANDOM_32_CHAR_STR

Paste your random string in place of “CHANGE_ME_TO_A_RANDOM_32_CHAR_STR”.

Then find: DB_PASSWORD=CHANGE_ME_DB_PASSWORD

Replace “CHANGE_ME_DB_PASSWORD” with a strong password (16+ characters, avoid # < > = &). Save.

Now open .db.env.

Find: MYSQL_PASSWORD=CHANGE_ME_DB_PASSWORD
Paste the SAME password you used in .env. They have to match exactly. If
they don’t match, the app cannot log into the database and the deployment
will fail. Save.

Step 6 — Launch the containers

From inside the CNM260 folder:

docker compose up -d — pull=always
What this does:

— pull=always 
Downloads the newest fireflyiii/core and mariadb:lts images

up 
Creates the network, the volumes, starts the containers and

containers -d
Detaches (runs in background, frees your terminal)

The first run takes about 60 seconds. Docker downloads roughly 400 MB of
images, MariaDB initializes its data directory, and Firefly III runs its
database migrations.

Confirm everything is running:

docker compose ps
You should see two services with status “Up” or “running”

fireflyiii
mariadb/

Step 7 — Open the app

Go to http://localhost:8080 in your browser. You’ll see the Firefly III
registration screen. Click Register. The first account you create
automatically becomes the admin.

After registering, you’ll go through the onboarding wizard — set your
default currency and create your first asset account.

Common problems and fixes
“port is already allocated” / “Bind for 0.0.0.0:8080 failed”
Something else on your machine is using port 8080. Edit
docker-compose.yml and change “8080:8080” to “9090:8080” (or any free
port). Then “docker compose up -d” again. Use http://localhost:9090.

“Access denied for user ‘firefly’@’%’”
MYSQL_PASSWORD in .db.env doesn’t match DB_PASSWORD in .env.

The catch:
MariaDB only sets the password the FIRST time it starts and writes it
to the volume. Changing the file later doesn’t change the database.
Fix: “docker compose down -v” (destroys data!), fix both files to match,
then “docker compose up -d — pull=always” again.

“App key is not set”
You forgot to fill in APP_KEY=, or there’s a forbidden character
(# < > = &) in it. Regenerate the key and try again.

Login page loads but credentials don’t work
You probably regenerated APP_KEY after creating your first user. The
APP_KEY encrypts session and password data, so changing it breaks them.
On a fresh install, the easiest fix is to wipe and start over:

docker compose down -v
docker compose up -d — pull=always
Full troubleshooting list is in troubleshooting.txt in the GitHub repo.

Day-to-day commands
Stop the deployment (data preserved):

docker compose down
Start it again:

docker compose up -d
Stop AND wipe all data:

docker compose down -v
View live app logs:

docker compose logs -f app
Update to the newest images:

docker compose pull && docker compose up -d
Back up the database:

docker exec firefly_iii_db mariadb-dump -u firefly -p firefly > backup.sql
Security notes if you go beyond localhost
This template is secure by default for localhost-only use. If you want to
expose Firefly III to the open internet, like I myself, plan to do soon?
Do these things first:

Put it behind a reverse proxy that terminates TLS (Caddy is easiest).
Set COOKIE_SECURE=true in .env.
Don’t expose port 3306. The compose file already keeps the database on
the internal Docker network — that’s intentional.
Back up the database volume on a schedule.
Pin image versions for production.
Never commit .env or .db.env files. The .gitignore in this repo
already handles that.
Wrapping up
You now have a self-hosted personal finance app at http://localhost:8080
with all your data stored in a Docker volume on your own machine.

The full template

docker-compose.yml
env examples
deployment guide
troubleshooting reference text
It all lives over at my repo: https://github.com/GAustin355/CNM260