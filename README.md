# bizmind-ai

AI-powered proposal generator for freelancers & agencies (FYP)

## Overview

This repository is a full-stack setup with a Rails API backend and a React (Vite) frontend.

- `backend/`: Rails 7 API with PostgreSQL, JWT authentication (Devise + devise-jwt), JSON serialization via ActiveModelSerializers, CORS enabled.
- `frontend/`: React 18 app using Vite, configured to run on port 3000.

Ports and CORS:
- Backend: http://localhost:3001 (Puma)
- Frontend: http://localhost:3000 (Vite)
- CORS allows requests from http://localhost:3000 and exposes the Authorization header.

## What’s included (implemented changes)

Backend (Rails API with JWT auth):
- Gems installed and configured:
	- devise (authentication)
	- devise-jwt (JWT token auth with denylist revocation)
	- active_model_serializers (JSON serialization)
	- rack-cors (CORS handling)
- Puma set to port 3001 by default (`backend/config/puma.rb`).
- CORS configured in `backend/config/initializers/cors.rb` to allow `http://localhost:3000`, expose `Authorization`, and allow credentials.
- Devise installed with JWT configuration in `backend/config/initializers/devise.rb`:
	- Issues JWT on `POST /api/v1/login` and `POST /api/v1/signup`
	- Revokes JWT on `DELETE /api/v1/logout`
	- Token expiration: 1 day
- Models and migrations:
	- `User` model generated via Devise and configured with `:jwt_authenticatable` using a denylist strategy
	- `JwtDenylist` model/table with indexed `jti` and `exp`
- Namespaced API routes (`api/v1`) in `backend/config/routes.rb` with custom paths:
	- Signup: `POST /api/v1/signup`
	- Login: `POST /api/v1/login`
	- Logout: `DELETE /api/v1/logout`
- Controllers:
	- `Api::V1::RegistrationsController` for signup (JSON responses)
	- `Api::V1::SessionsController` for login/logout (JSON responses)
	- `Api::V1::BaseController` requiring `authenticate_user!` for protected endpoints
	- `ApplicationController` responds to JSON
- Serialization:
	- `UserSerializer` returning `id`, `email`, `created_at`, `updated_at`

Frontend (React + Vite):
- Vite server set to port 3000 with `strictPort: true` and `open: true` (`frontend/vite.config.js`).
- Note: Vite 7 requires Node.js 20.19+ or 22.12+.

## Prerequisites

- Ruby 3.2.x and Bundler
- PostgreSQL running and accessible (configure `backend/config/database.yml` as needed)
- Node.js 20.19+ (required by Vite 7) and npm

## Local development

Backend (Rails API):

```bash
cd backend
bundle install
bin/rails db:create db:migrate  # ensure Postgres is running and database.yml is configured
bin/rails s -p 3001
```

Frontend (Vite React):

```bash
cd frontend
npm install
npm run dev
```

The frontend is served at http://localhost:3000 and the backend at http://localhost:3001.

## API: Authentication

Headers (for JSON requests):
- `Content-Type: application/json`
- For protected requests: `Authorization: Bearer <JWT>`

Auth endpoints (JSON):
- Signup: `POST /api/v1/signup`
	- Body:
		```json
		{
			"user": {
				"email": "test@example.com",
				"password": "password123",
				"password_confirmation": "password123"
			}
		}
		```
	- Response: `201 Created` with `{ user: { ... } }` and `Authorization: Bearer <token>` header

- Login: `POST /api/v1/login`
	- Body:
		```json
		{
			"user": {
				"email": "test@example.com",
				"password": "password123"
			}
		}
		```
	- Response: `200 OK` with `{ user: { ... } }` and `Authorization: Bearer <token>` header

- Logout: `DELETE /api/v1/logout`
	- Header: `Authorization: Bearer <token>`
	- Response: `204 No Content` (token revoked)

Health check:
- `GET /up` → `200 OK` if the app boots successfully

### cURL examples

Signup:
```bash
curl -i -X POST http://localhost:3001/api/v1/signup \
	-H 'Content-Type: application/json' \
	-d '{
		"user": {
			"email": "test@example.com",
			"password": "password123",
			"password_confirmation": "password123"
		}
	}'
```

Login:
```bash
curl -i -X POST http://localhost:3001/api/v1/login \
	-H 'Content-Type: application/json' \
	-d '{
		"user": {
			"email": "test@example.com",
			"password": "password123"
		}
	}'
```

Logout:
```bash
curl -i -X DELETE http://localhost:3001/api/v1/logout \
	-H 'Authorization: Bearer <PASTE_TOKEN_HERE>'
```

## Implementation details (code map)

- CORS: `backend/config/initializers/cors.rb`
- Puma port: `backend/config/puma.rb`
- Devise + JWT setup: `backend/config/initializers/devise.rb`
- User model: `backend/app/models/user.rb`
- JWT denylist: `backend/app/models/jwt_denylist.rb`, migration in `backend/db/migrate/*create_jwt_denylists.rb`
- Routes: `backend/config/routes.rb` (namespaced `api/v1` with Devise controllers)
- Controllers:
	- `backend/app/controllers/api/v1/registrations_controller.rb`
	- `backend/app/controllers/api/v1/sessions_controller.rb`
	- `backend/app/controllers/api/v1/base_controller.rb`
	- `backend/app/controllers/application_controller.rb`
- Serialization: `backend/app/serializers/user_serializer.rb`
- Frontend dev server config: `frontend/vite.config.js`

## Notes & troubleshooting

- Node < 20.19 will print an engine warning and Vite may refuse to start; upgrade Node to 20.19+ or 22.12+.
- If port 3000 or 3001 is occupied, update:
	- Vite port in `frontend/vite.config.js`
	- Puma port in `backend/config/puma.rb`
	- CORS origins in `backend/config/initializers/cors.rb`
- Ensure PostgreSQL is running and accessible; adjust `backend/config/database.yml` if needed.
