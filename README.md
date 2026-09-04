# Expense Tracker

A classroom-ready MERN expense tracker with a cyber-inspired React dashboard, an Express REST API, and MongoDB persistence.

## Application Structure

```text
.
├── README.md
├── backend/
│   ├── Dockerfile.backend
│   ├── package.json
│   └── src/
│       ├── server.js
│       ├── models/Expense.js
│       └── routes/expenses.js
└── frontend/
    ├── Dockerfile.frontend
    ├── package.json
    ├── index.html
    ├── nginx/default.conf
    └── src/
        ├── main.jsx
        └── styles.css
```

## Features

- Add, edit, and delete expenses.
- Search titles and filter by category or inclusive date range.
- View total spend, transaction count, recent expenses, and category totals.
- Seed four demo expenses when the database is empty.
- Export the currently visible expenses to `expenses.csv` in the browser.

Supported categories are `Food`, `Transport`, `Shopping`, `Bills`, `Education`, `Health`, `Travel`, and `Other`.

## Requirements

- Node.js 20 or later
- npm
- MongoDB
- Docker

The backend uses `mongodb://localhost:27017/expense_tracker` by default. You can override it with a `.env` file in `backend/`:

```text
NODE_ENV=development
PORT=5000
MONGO_URI=mongodb://localhost:27017/expense_tracker
CORS_ORIGIN=http://localhost:5173
```

## Run Locally

Start MongoDB, then open two terminals.

Start the API:

```bash
cd backend
npm install
npm start
```

The backend listens on port `5000`.

Start the React development server:

```bash
cd frontend
npm install
npm run dev
```

The frontend requests `/api` from its own origin. For local development, configure your development server to proxy `/api` to `http://localhost:5000`; the current Vite setup does not include a proxy configuration.

To create a production frontend build:

```bash
cd frontend
npm run build
npm run preview
```

## Run with Docker

Build both images from the project root:

```bash
docker build -f backend/Dockerfile.backend -t exp-backend:v1 ./backend
docker build -f frontend/Dockerfile.frontend -t exp-frontend:v1 ./frontend
```

Create the Docker network used by the containers:

```bash
docker network create exp-network
```

Start MongoDB:

```bash
docker run -d \
  --network=exp-network \
  --name mongo \
  mongo:7
```

Start the backend. The `backend` container name is required by the frontend NGINX proxy:

```bash
docker run -d \
  -p 5000:5000 \
  --network exp-network \
  --name backend \
  -e "MONGO_URI=mongodb://mongo:27017/expense_tracker" \
  exp-backend:v1
```

Check the backend logs and health endpoint:

```bash
docker logs backend
curl http://192.168.120.189:5000/api/health
```

Replace `192.168.120.189` with the Docker host IP when accessing the API remotely. From the same host, `curl http://localhost:5000/api/health` also works.

Start the frontend on host port `9155`:

```bash
docker run -d -it \
  -p 9155:80 \
  --network exp-network \
  --name frontend-main \
  exp-frontend:v1
```

Open the dashboard at:

```text
http://192.168.120.189:9155
```

Replace the IP with the Docker host IP. The frontend proxies `/api` requests to the `backend` container over `exp-network`.

Useful Docker commands:

```bash
docker ps
docker network inspect exp-network
docker logs backend
docker logs frontend-main
docker stop frontend-main backend mongo
docker rm frontend-main backend mongo
docker network rm exp-network
```

## API

The backend base URL is `http://localhost:5000/api`. Successful responses use the shape `{ "success": true, "data": ... }`.

### Health check

```bash
curl http://localhost:5000/api/health
```

Returns API status, MongoDB connection state, and a timestamp.

### List expenses

```bash
curl 'http://localhost:5000/api/expenses'
curl 'http://localhost:5000/api/expenses?category=Food&search=coffee&from=2026-01-01&to=2026-01-31'
```

Query parameters:

- `category`: one supported category; `All` leaves the category unfiltered.
- `search`: case-insensitive title search.
- `from`: inclusive start date, `YYYY-MM-DD`.
- `to`: inclusive end date, `YYYY-MM-DD`.

Results are sorted newest first and include `count` and `data`.

### Summary

```bash
curl 'http://localhost:5000/api/expenses/summary?from=2026-01-01&to=2026-01-31'
```

Returns `total`, `count`, `byCategory`, and the five most recent matching expenses. The summary accepts date filters; category and title filters apply only to the list endpoint.

### Create an expense

```bash
curl -X POST http://localhost:5000/api/expenses \
  -H 'Content-Type: application/json' \
  -d '{"title":"Coffee","amount":4.50,"category":"Food","note":"Morning coffee","spentAt":"2026-01-15"}'
```

Required fields are `title`, `amount`, and `category`. `note` and `spentAt` are optional; the date defaults to now. Titles are limited to 120 characters, notes to 300 characters, and amounts must be non-negative.

### Update or delete an expense

Replace `EXPENSE_ID` with the MongoDB document `_id` returned by the API:

```bash
curl -X PUT http://localhost:5000/api/expenses/EXPENSE_ID \
  -H 'Content-Type: application/json' \
  -d '{"amount":6.25,"note":"Updated note"}'

curl -X DELETE http://localhost:5000/api/expenses/EXPENSE_ID
```

### Seed demo data

```bash
curl -X POST http://localhost:5000/api/expenses/seed
```

Seeding is skipped when any expense already exists. Otherwise, it creates Team lunch, AWS EC2 lab, Metro ride, and Internet bill records.

## Production Hosting

Build the frontend with `npm run build` and serve its generated `dist/` directory from a static web server. The server must proxy `/api` requests to the running backend at `http://localhost:5000` or another configured backend host. Keep MongoDB private and provide the backend with a production `MONGO_URI`.

For a real deployment, also add HTTPS, authentication, secret management, database backups, monitoring, and a managed MongoDB service such as MongoDB Atlas.

## Troubleshooting

### MongoDB connection fails

Confirm that MongoDB is running and that `MONGO_URI` points to the correct database. The backend exits during startup if it cannot connect.

### The frontend cannot reach the API

Confirm that the `backend` container is running on port `5000`, that both application containers are connected to `exp-network`, and that the frontend is running on port `9155`. Directly test the API with:

```bash
curl http://localhost:5000/api/health
```

### Reset Docker containers

```bash
docker stop frontend-main backend mongo
docker rm frontend-main backend mongo
docker network rm exp-network
```
