# Expense Tracker — MERN + Docker Compose

A classroom-ready **Expense Tracker MERN app** built with:

- **MongoDB** — database
- **Express.js** — REST API
- **React + Vite** — frontend
- **NGINX** — serves the frontend and proxies `/api` to the backend
- **Docker Compose** — runs the full stack with one command

The UI uses a cyber AI inspired brand style: dark navy background, purple gradients, neon cyan highlights, and glass panels.

---

## Architecture

```text
Browser
  |
  v
Frontend Container: React static files served by NGINX
  |
  | /api requests are proxied by NGINX
  v
Backend Container: Express.js API
  |
  v
MongoDB Container: persistent database volume
```

---

## Project Structure

```text
expense-tracker-mern/
│
├── docker-compose.yml
├── .env.example
├── README.md
│
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
│       ├── server.js
│       ├── models/
│       │   └── Expense.js
│       └── routes/
│           └── expenses.js
│
└── frontend/
    ├── Dockerfile
    ├── package.json
    ├── index.html
    ├── nginx/
    │   └── default.conf
    └── src/
        ├── main.jsx
        └── styles.css
```

---

## Features

- Add expense
- Edit expense
- Delete expense
- Filter by category
- Search by title
- Filter by date range
- Summary cards
- Category breakdown bars
- Seed demo data
- Export visible expenses as CSV
- Dockerized frontend, backend, and MongoDB

---

# Run Locally with Docker Compose

From the project root:

```bash
docker compose up -d --build
```

Open the app:

```text
http://localhost
```

Check containers:

```bash
docker compose ps
```

Check logs:

```bash
docker compose logs -f
```

Stop the app:

```bash
docker compose down
```

Stop and delete database volume:

```bash
docker compose down -v
```

---

# API Endpoints

Health check:

```bash
curl http://localhost/api/health
```

Get expenses:

```bash
curl http://localhost/api/expenses
```

Seed demo data:

```bash
curl -X POST http://localhost/api/expenses/seed
```

Create expense:

```bash
curl -X POST http://localhost/api/expenses \
  -H "Content-Type: application/json" \
  -d '{"title":"Coffee","amount":4.5,"category":"Food","note":"Morning coffee"}'
```

---

# AWS EC2 Deployment Instruction

## 1. Create EC2 Instance

Recommended settings for classroom demo:

```text
AMI: Ubuntu Server 22.04 LTS or Ubuntu Server 24.04 LTS
Instance type: t2.micro / t3.micro for demo, t3.small preferred
Storage: 12 GB or more
```

## 2. Configure Security Group

Inbound rules:

```text
SSH    22   Your IP only
HTTP   80   0.0.0.0/0
```

Optional for debugging only:

```text
Custom TCP 5000   Your IP only
Custom TCP 27017  Your IP only, generally avoid exposing MongoDB publicly
```

For this app, the browser only needs port `80` because NGINX proxies API calls internally.

---

## 3. SSH into EC2

From your laptop:

```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

---

## 4. Install Docker and Docker Compose Plugin

Run these commands on the EC2 Ubuntu machine:

```bash
sudo apt update
sudo apt install -y ca-certificates curl unzip
```

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo \"${UBUNTU_CODENAME:-$VERSION_CODENAME}\") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Allow the `ubuntu` user to run Docker:

```bash
sudo usermod -aG docker ubuntu
exit
```

SSH again:

```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

Verify Docker:

```bash
docker --version
docker compose version
```

---

## 5. Upload Project ZIP to EC2

From your laptop, from the folder where the zip exists:

```bash
scp -i your-key.pem expense-tracker-mern.zip ubuntu@YOUR_EC2_PUBLIC_IP:/home/ubuntu/
```

SSH into EC2:

```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

Unzip:

```bash
unzip expense-tracker-mern.zip
cd expense-tracker-mern
```

---

## 6. Start the Full MERN App

```bash
docker compose up -d --build
```

Check status:

```bash
docker compose ps
```

Check logs:

```bash
docker compose logs -f
```

Open in browser:

```text
http://YOUR_EC2_PUBLIC_IP
```

---

## 7. Test the API on EC2

```bash
curl http://localhost/api/health
```

Seed demo data:

```bash
curl -X POST http://localhost/api/expenses/seed
```

Refresh the browser.

---

## 8. Useful Docker Compose Commands for Students

Show running containers:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f backend
```

Restart one service:

```bash
docker compose restart backend
```

Rebuild after code changes:

```bash
docker compose up -d --build
```

Stop containers:

```bash
docker compose down
```

Stop and remove MongoDB data volume:

```bash
docker compose down -v
```

---

## 9. Go Inside Containers

Frontend container:

```bash
docker exec -it expense-frontend sh
ls /usr/share/nginx/html
cat /etc/nginx/conf.d/default.conf
exit
```

Backend container:

```bash
docker exec -it expense-backend sh
ls
node -v
exit
```

MongoDB container:

```bash
docker exec -it expense-mongo mongosh expense_tracker
show collections
db.expenses.find().pretty()
exit
```

---

## 10. Classroom Explanation

```text
Dockerfile builds one service image.
Docker Compose starts the complete application stack.
```

For this project:

```text
frontend service:
  React app is built and served by NGINX.

backend service:
  Express API connects to MongoDB.

mongo service:
  Official MongoDB image stores data in a Docker volume.
```

Best analogy:

```text
Dockerfile = how to build one room
Docker Compose = how to run the entire house with all rooms connected
```

---

## 11. Troubleshooting

### Port 80 already in use

Check what is using port 80:

```bash
sudo lsof -i :80
```

Stop Apache/NGINX if installed on the host:

```bash
sudo systemctl stop apache2 || true
sudo systemctl stop nginx || true
```

Run Compose again:

```bash
docker compose up -d
```

### Frontend opens but API fails

Check backend logs:

```bash
docker compose logs backend
```

Check Mongo logs:

```bash
docker compose logs mongo
```

### Rebuild cleanly

```bash
docker compose down
docker compose up -d --build
```

### Reset all app data

```bash
docker compose down -v
docker compose up -d --build
```

---

## Production Notes

For a real production deployment, add:

- HTTPS with NGINX reverse proxy or AWS ALB + ACM certificate
- Authentication and authorization
- MongoDB Atlas or managed database
- CI/CD pipeline
- Environment-specific secrets
- Backups for database volume
- Monitoring and log collection
