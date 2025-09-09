# DeRisk Starknet

## Projects overview
This project consist of a monorepo with components required for the implementation of DeRisk on Starknet.
There are several components in this repository, each with its own purpose and functionality. The main components are:

### Shared
[`shared`](./apps/shared/README.md)
Both projects [`data_handler`](./apps/data_handler/README.md), [`dashboard_app`](./apps/dashboard_app/README.md)  work with the same shared database,
which is contained in the shared folder. It also contains all shared modules.

### Data Handler
[`data_handler`](./apps/data_handler/README.md)
Data processing and analysis component: Collects data from DeFi, analyzes it, and saves it to the db. It contains Celery tasks to schedule data collection runs. Once the data is collected, it triggers an endpoint on the `dashboard_app`

### Dashboard App 
[`dashboard_app`](./apps/dashboard_app/README.md)
Works as a server for the [`frontend_dashboard`](./apps/frontend_dashboard/README.md). Generates an analytics dashboard using Streamlit. Contains an API to handle the Telegram webhook and send bot messages.
Key Features:
- Interactive data visualization
- Protocol statistics monitoring
- Loan portfolio analysis
- Real-time data updates


### Dashboard Frontend 
[`frontend_dashboard`](./apps/frontend_dashboard/README.md)
React client app. Uses API if the  [`dashboard_app`](./apps/dashboard_app/README.md)


## Legacy Apps
Legacy apps contain code that has been migrated to the current apps, along with additional code. Explore them in case of a stack issue:
- [web_app](https://github.com/CarmineOptions/derisk-research/tree/5a4b9dc752399480f41884a286ab28b2b9804468/apps/web_app)
- [legacy_app](https://github.com/CarmineOptions/derisk-research/tree/5a4b9dc752399480f41884a286ab28b2b9804468/apps/legacy_app)
- [sdk](https://github.com/CarmineOptions/derisk-research/tree/5a4b9dc752399480f41884a286ab28b2b9804468/apps/sdk)



## Quick Start Guide

### Prerequisites
- Docker installed on your machine (v19.03+ recommended).
- Docker Compose installed (v2.0+ recommended).

### Data Handler local development

1. To set up this project run next command for local development in `derisk-research` directory:

2. Environment Configuration:
```bash
cp apps/data_handler/.env.dev apps/data_handler/.env
```
3. Start the Services:

```bash
docker-compose -f devops/dev/docker-compose.data-handler.yaml up --build
```
4. Stop the Services:
```bash
docker-compose -f devops/dev/docker-compose.data-handler.yaml down
```

5. To run test cases for this project run next command in `derisk-research` directory:
```bash
make test_data_handler
```

For detailed documentation, see the [Data Handler](./apps/data_handler/README.md)




### Dashboard App local development
1. To set up this project run next command for local development in `derisk-research` directory:

2. Environment Configuration:
```bash
cp apps/dashboard_app/.env.dev apps/dashboard_app/.env
```
3. Start the Services:

```bash
docker-compose -f devops/dev/docker-compose.dashboard-app.yaml up --build
```
4. Stop the Services:
```bash
docker-compose -f devops/dev/docker-compose.dashboard-app.yaml down
```

#### Backend API (FastAPI)

```bash
cd apps/dashboard_app
poetry install
cp .env.dev .env  
poetry run uvicorn app.main:app --reload --port 8000
```

Explore the API will be available at http://localhost:8000/docs. 

#### Frontend App (React + Vite)

Make sure `dashboard_app` is running!

```bash
cd apps/frontend_dashboard
npm install
npm run dev
```

Navigate to http://localhost:5173 to access the subscription form.

#### CORS & Proxy
The backend is configured with CORS to allow requests from http://localhost:5173, and the Vite dev server proxies `/api` to the backend.

## Shared package (Common code shared between the components)
1. How to run test cases for shared package, run next command in root folder:
```bash
make test_shared
```

## Running the dashboard frontend with Docker

1. **Naviagte to frontend_dashboard directory**:
   ```bash
   cd apps/frontend_dashboard
   ```
2. **Build the Docker Image**:
   ```bash
   docker build -t frontend_dashboard .
   ```

   or on linux

   ```bash
   sudo docker build -t frontend_dashboard .
   ```

3. **Run the Docker Container:**:

    ```bash
    docker run -p 5173:5173 frontend_dashboard
    ```

     or on linux

   ```bash
   sudo docker run -p 5173:5173 frontend_dashboard
   ```
4. **Access the Application**:
    Open your browser and navigate to `http://localhost:5173`.





## Data migrations with Alembic
In this project is using alembic for data migrations.

### Generating Migrations
Navigate to the project folder and generate a new migration using the following command:
1. Run:
```bash
docker compose -f devops/dev/docker-compose.db.yaml up --build
```
2. Then:
```bash
cd apps/shared
alembic -c alembic.ini revision --autogenerate -m "your message"
```

3. ? You may need to set up `.env` and install missing packages. Check the terminal, as pip-install missing packages


### Applying Migrations
After generating new migration, you need to apply it:

```bash
alembic -c alembic.ini upgrade head
```
### Rolling Back Migrations
For downgrading migration:

```bash
alembic -c alembic.ini downgrade -1
```

### Migration Utility Commands
Useful commands:
Purge all celery tasks:
```bash
docker compose run --rm celery celery -A celery_conf purge
```
Purge all celery beat tasks:
```bash
docker compose run --rm celery_beat celery -A celery_conf purge
```
Go to bash
```bash
docker compose exec backend bash
```


#TODO review:
## How to run migration command:
1. Set up `.env.dev` into `derisk-research/apps/data_handler`
2. Go back to `derisk-research/apps` directory
3. Then run bash script to migrate:
```bash
bash data_handler/migrate.sh
```