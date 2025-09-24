# bizmind-ai

## Development setup

This repo contains two apps:

- `backend/`: Rails 7 API (PostgreSQL) running on port 3001
- `frontend/`: React 18 + Vite running on port 3000

### Prerequisites

- Ruby 3.2.x and Bundler
- PostgreSQL running locally and accessible (configure `backend/config/database.yml` as needed)
- Node.js 20.19+ (required by Vite 7) and npm

### Install and run

Backend (Rails API):

```bash
cd backend
bundle install
bin/rails db:create db:migrate # ensure Postgres is running and database.yml is configured
bin/rails s -p 3001
```

Frontend (Vite React):

```bash
cd frontend
npm install
npm run dev
```

The frontend is configured to run at http://localhost:3000 and the backend at http://localhost:3001. CORS is enabled in `backend/config/initializers/cors.rb` to allow requests from the frontend during development.

### Notes

- If Node is < 20.19, upgrade Node to satisfy Vite 7 engine requirements.
- If port 3000 or 3001 is in use, adjust Vite `vite.config.js` and Puma `config/puma.rb` accordingly and update CORS origins.
AI-powered proposal generator for freelancers &amp; agencies (FYP)
