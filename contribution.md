# Team Contribution

**Project:** Dublin Bikes Full-Stack Web Application  
**Team Members:** Kaiwen Yao · Alex · Zeline

---

## Overview

## Kaiwen Yao

### Scraper (`scraper/`)
- Designed and implemented the standalone data collection service
- `main_scraper.py`: periodic scraping loop with configurable interval and retry logic
- `fetch_stations.py`: pulls real-time station and bike availability data from the JCDecaux API
- `fetch_weather.py`: fetches weather data for Dublin
- `models.py` / `models_weather.py`: SQLAlchemy ORM models for `station`, `availability`, and weather tables
- Containerised with Docker; Jenkins CI/CD pipeline for automated image build and EC2 deployment

### Flask Backend – Project Foundation (`flask-app/`)
- Initialised Flask application factory pattern (`app/__init__.py`, `extensions.py`)
- Configured environment-based settings (`config.py`) with strict validation on startup
- Set up Flask-Migrate for database schema management; migrations shared with scraper
- Wrote `wsgi.py` and `entrypoint.sh` for production Gunicorn deployment

### Flask Backend – User Authentication API (`app/api/user_routes.py`, `app/services/user_service.py`)
- Full JWT-based authentication: registration, login, logout, access/refresh token rotation
- Email verification flow: send verification code, activate by code, activate by token link
- Pydantic request/response contracts (`app/contracts/`)
- Password hashing, token expiry, and auth error handling (`AuthError`)

### Flask Backend – Station API (`app/api/station_routes.py`, `app/services/station_service.py`)
- `GET /api/stations/` – list all stations
- `GET /api/stations/<number>/availability` – recent 24 h availability records
- `GET /api/stations/status` – latest snapshot across all stations
- `GET /api/stations/<number>/predict` – bike availability prediction (delegates to prediction service)

### Machine Learning & Prediction Service (`machine_learning/`, `app/services/prediction_service.py`)
- Conducted full ML pipeline in `ml.ipynb`: EDA, feature engineering, correlation analysis
- Trained and compared Linear Regression, Decision Tree, Random Forest, and Gradient Boosting models
- Selected Decision Tree as best-performing model; exported `bike_availability_model.pkl` and `model_features.pkl`
- Published model to Hugging Face (`ucdse/bike_availability_model`); Jenkins pipeline pulls model at build time
- Implemented `prediction_service.py` to load and serve predictions at runtime

### Flask Backend – AI Chat API (`app/api/chat_routes.py`, `app/services/chat_service.py`)
- Architected a full conversational AI backend integrating Alibaba Cloud Qwen LLM via LangChain, enabling context-aware multi-turn dialogue
- Implemented dual response modes: standard JSON (`POST /api/chat`) and real-time SSE streaming (`POST /api/chat/stream`) using Flask's `stream_with_context` for low-latency token-by-token output
- Built persistent session management with `generate_session_id`, `get_session_messages`, and a `message_store` table to maintain conversation history across requests
- Secured all chat endpoints with JWT middleware, ensuring only authenticated users can access the LLM

### Scrum Master (Sprint 3)
- Facilitated sprint planning, stand-ups, and retrospective for Sprint 3, coordinating backend integration milestones and ensuring timely resolution of blockers

### Testing & CI/CD
- Built the pytest test infrastructure (`tests/conftest.py`): SQLite in-memory database isolation, shared fixtures for users, stations, availability, weather, and auth headers
- Wrote unit and integration tests for user auth, station API, prediction service, contracts, schemas, utilities, and AI chat (`test_user_routes.py`, `test_user_service.py`, `test_station_routes.py`, `test_station_service.py`, `test_prediction_service.py`, `test_contracts.py`, `test_schemas.py`, `test_utils.py`, `test_email_utils.py`, `test_chat_routes.py`, `test_chat_service.py`, `test_chat_service_llm.py`)
- Wrote Jenkins pipeline (`Jenkinsfile`): syntax check → test → ML model download → Docker build/push → EC2 deploy

---

## Alex

### React App Architecture
- Bootstrapped the React + TypeScript + Vite project
- Configured routing with `react-router-dom` (`src/router/index.tsx`)
- Set up Tailwind CSS, ESLint, and project-wide TypeScript config

### Layout & Shared Components (`src/components/`)
- `Layout.tsx` – top-level layout wrapper with outlet
- `Header.tsx` – site-wide navigation bar
- `Footer.tsx` – site-wide footer

### Maps Page
- Integrated Google Maps JavaScript API for interactive bike station map
- Displayed real-time station markers with availability badges
- Implemented station detail panel showing current and historical availability

### Home Page (`src/pages/Home/Home.tsx`)
- Landing page with project introduction and quick navigation

### Authentication Pages
- `Register.tsx` – registration form with client-side validation
- `Login.tsx` – login form with JWT token storage
- `VerifyEmail.tsx` – prompt page after registration
- `Activate.tsx` – email token activation handler

### Profile Page
- Displays user info retrieved from `/api/users/me`
- Supports profile update and logout

### User Research – Persona Interviews & Script Design
- Designed and conducted user persona interviews to identify target user groups and core usage scenarios for the Dublin Bikes application
- Authored the full interview script covering user commuting habits, pain points with existing bike-share services, and feature expectations
- Synthesised interview findings into user personas that informed the frontend UX priorities and feature scope

### Scrum Master (Sprint 1 & Sprint 4)
- Facilitated sprint planning, daily stand-ups, and retrospectives across Sprint 1 and Sprint 4, keeping the team aligned on delivery goals and unblocking cross-functional dependencies
- Maintained and prioritised the product backlog, broke down user stories into actionable tasks, and tracked sprint velocity to ensure on-time delivery
- Acted as the primary communication bridge between team members, coordinating integration points between frontend and backend to prevent merge conflicts and interface mismatches

### Flask Backend – Real-Time Station Status API
- Designed and implemented `GET /api/stations/status` – a high-performance endpoint that returns the latest real-time availability snapshot across all stations in a single request
- Engineered an efficient SQLAlchemy subquery using `func.max()` grouped by station number to retrieve only the most recent record per station, avoiding full-table scans and ensuring consistent response times regardless of historical data volume
- Ensured the response format aligns precisely with the existing team API contract by reusing the shared `_availability_to_dict` serialiser, maintaining zero-friction integration with the frontend map layer

---

## Zeline

### UI/UX Design – Webpage Mockups
- Designed high-fidelity mockups for all major pages including Home, Maps, Chat, Profile, and authentication flows using Figma
- Defined the visual language of the application: colour palette, typography, component hierarchy, and responsive layout grid
- Produced annotated wireframes that served as the blueprint for frontend implementation, ensuring design-to-code consistency across the team

### Scrum Master (Sprint 2)
- Facilitated sprint planning, stand-ups, and retrospective for Sprint 2, managing task prioritisation and cross-team coordination during the core feature development phase

### AWS RDS – Database Setup & Integration
- Provisioned and configured an Amazon RDS MySQL instance as the production database, including security group rules, subnet configuration, and connection parameter tuning
- Managed database credentials and connection strings via environment variables, ensuring secure and environment-agnostic connectivity across local, Docker, and EC2 deployments
- Validated schema compatibility between Flask-Migrate migrations and the RDS instance, coordinating the initial `flask db upgrade` on the production environment

### Flask Backend – Weather API (`app/api/weather_routes.py`, `app/services/weather_service.py`)
- `GET /api/weather` – fetches hourly and daily forecast for Dublin from OpenWeatherMap One Call API
- Pydantic `WeatherQueryDTO` / `WeatherDataVO` for strict request validation and response serialisation
- Error handling for invalid coordinates and API failures (`WeatherAPIError`)

### Flask Backend – Journey / Route Planning API (`app/api/journey_routes.py`, `app/services/journey_service.py`)
- `POST /api/journey/plan` – accepts either text addresses or coordinates
- Google Maps Geocoding integration to resolve address strings to coordinates
- Server-side optimal route calculation: nearest available source station → nearest free-dock destination station
- `api_retry` decorator for resilient Google Maps API calls

### React – Weather Component
- Reusable weather widget consuming `/api/weather`
- Displays current conditions, hourly forecast, and daily forecast
- Used across Maps and Home pages

### React – Chat Page
- Full chat UI with message history, typing indicators, and SSE streaming display
- Session persistence across page reloads
- Connected to both `/api/chat` and `/api/chat/stream` endpoints

### React – Journey Planning UI (within `Maps.tsx`)
- Journey search form (start / end address or coordinates)
- Renders recommended pick-up and drop-off stations on the map
- Displays estimated route duration from the journey planning API

### Testing – Weather & Journey APIs
- Wrote unit and integration tests for all self-implemented backend modules: weather and journey
- `test_weather_routes.py`, `test_weather_routes_validation.py`, `test_weather_service.py`
- `test_journey_routes.py`, `test_journey_service.py`, `test_journey_service_matrix.py`
- Mocked external dependencies: Google Maps API
