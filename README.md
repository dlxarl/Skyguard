# SkyGuard

**A community-powered aerial threat awareness and alert platform.**

SkyGuard helps people share reports of potential aerial threats, view reported incidents on a live map, locate nearby shelters, and receive location-based notifications through Telegram.

The project was built for **Hack4Defence**. It combines a React map interface, a Django REST API, community-submitted reports, and Telegram messaging to make local threat information easier to access.

> **Important:** SkyGuard is an informational and community-reporting tool. It is not an official emergency alert system, does not verify threats independently, and must never replace instructions from competent authorities or official warning channels. Always follow official guidance and move to safety when an alert is issued.

## Key capabilities

- **Live threat map** — displays reported aerial threats and shelter locations on an interactive map.
- **Community reporting** — authenticated users can submit a report with their current GPS coordinates and select the reported object type.
- **Report aggregation** — nearby reports of the same type can be grouped to reduce duplicate map entries and combine reports from different users.
- **Threat confidence indicators** — aggregated reports have a probability level based on the report count and contributor trust scores.
- **Shelter directory** — displays available shelter records, including their coordinates, address, and capacity when provided.
- **Telegram alerts** — users can link a Telegram account and receive notifications about reported threats near their last known location.
- **User profiles** — profile details, avatars, notification preferences, and contributor trust ratings.
- **Moderation workflow** — privileged administrators can review pending reports and confirm or reject aggregated reports.
- **Safety guide** — includes guidance on recognizing reported object types and finding safer locations.

## How it works

1. A user signs in and submits a report from the map interface.
2. The browser requests the user's location and sends the coordinates and selected target type to the backend.
3. The backend stores the report and checks whether it can be associated with nearby reports of the same type.
4. Reports are presented on the map with their current moderation status and confidence information.
5. Users who have connected Telegram and enabled notifications can receive a message when a reported threat is within the configured notification radius.
6. Administrators can review aggregated reports and confirm or reject them. The moderation result updates contributor trust ratings.

The application currently uses a **30 km Telegram notification radius**. This is a software configuration, not a prediction of a threat's actual range or impact.

## Supported report types

| Type | Description |
| --- | --- |
| Drone | Report of an unmanned aerial vehicle |
| Rocket | Report of a rocket or missile |
| Plane | Report of an aircraft |
| Helicopter | Report of a helicopter |
| Bang | Report of an explosion or loud blast |

Reports can include an optional text description. The backend also stores a danger-radius value for a report.

## Technology stack

### Frontend

- React 19
- TypeScript
- Vite
- React Router
- Axios
- Leaflet
- React Leaflet

### Backend

- Python
- Django
- Django REST Framework
- Django REST Framework Simple JWT
- PostgreSQL
- `django-admin-geomap`
- `django-cors-headers`

### Integrations

- Telegram Bot API
- Browser Geolocation API

## Repository structure

```text
Skyguard/
├── backend/
│   ├── backend/          # Django project configuration and URL routing
│   ├── targets/          # Threat reports, aggregation, shelters and notifications
│   ├── users/            # Accounts, profiles, trust ratings and Telegram linking
│   ├── media/            # User-uploaded media
│   ├── manage.py
│   └── telegram_bot.py   # Telegram bot polling process
├── frontend/
│   ├── src/
│   │   ├── components/   # Map and shared UI components
│   │   ├── context/      # Application context
│   │   ├── pages/        # Home, authentication and safety guide
│   │   ├── api.ts        # Backend API client
│   │   └── types.ts      # Shared TypeScript types
│   └── package.json
├── requirements.txt
└── README.md
```

## Getting started

### Prerequisites

Install the following before running the application:

- Python 3.10 or newer
- Node.js 20 or newer and npm
- PostgreSQL
- A Telegram bot token from [@BotFather](https://t.me/BotFather) (needed for Telegram integration)

The commands below assume a local development environment and a PostgreSQL database.

### 1. Clone the repository

```bash
git clone https://github.com/dlxarl/Skyguard.git
cd Skyguard
```

### 2. Configure the backend

Create and activate a virtual environment:

```bash
cd backend
python -m venv .venv
```

On macOS or Linux:

```bash
source .venv/bin/activate
```

On Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Install the backend dependencies:

```bash
pip install -r ../requirements.txt
pip install Django djangorestframework djangorestframework-simplejwt django-cors-headers django-admin-geomap django-jazzmin psycopg2-binary Pillow requests
```

Create a `.env` file inside the `backend/` directory:

```dotenv
SECRET_KEY=replace-with-a-long-random-secret
DEBUG=True

DB_NAME=skyguard
DB_USER=postgres
DB_PASSWORD=your-database-password
DB_HOST=localhost
DB_PORT=5432

TELEGRAM_BOT_TOKEN=your-telegram-bot-token
```

Create a PostgreSQL database named `skyguard` and make sure the database credentials match the values in `.env`.

Apply migrations and create an administrator account:

```bash
python manage.py migrate
python manage.py createsuperuser
```

Start the API server:

```bash
python manage.py runserver
```

The backend will be available at `http://127.0.0.1:8000/`.

### 3. Start the frontend

Open a second terminal:

```bash
cd frontend
npm install
npm run dev
```

Vite will print the local development URL, normally `http://localhost:5173/`.

### 4. Configure Telegram

1. Create a bot with [@BotFather](https://t.me/BotFather).
2. Put the bot token in `backend/.env` as `TELEGRAM_BOT_TOKEN`.
3. Start the Django backend.
4. Start the bot polling process in another terminal:

```bash
cd backend
source .venv/bin/activate
python telegram_bot.py
```

5. Register or sign in to SkyGuard, open the profile section, and use **Connect Telegram**.
6. Open the generated Telegram link to link your account.
7. Allow location access in the application and keep notifications enabled.

The bot currently supports `/start`, `/status`, and `/help`. The project also contains a webhook-based account-linking endpoint; use one bot update delivery mode at a time (polling or webhook) when deploying.

## API overview

The backend is built with Django REST Framework. The main route groups are:

| Route | Purpose | Access |
| --- | --- | --- |
| `/api/targets/` | List active reports and submit a report | Public read; authenticated write |
| `/api/shelters/` | List shelter locations | Public |
| `/api/register/` | Create an account | Public |
| `/api/me/` | Read or update the current profile | Authenticated |
| `/api/me/avatar/` | Upload a profile avatar | Authenticated |
| `/api/me/location/` | Update the last known location | Authenticated |
| `/api/me/telegram/` | Configure Telegram notification settings | Authenticated |
| `/api/me/telegram/link/` | Generate a one-time Telegram linking code | Authenticated |

Authentication uses JWT. The exact route configuration is defined in the Django URL modules.

Administrative report review endpoints are restricted to Django staff users.

## Telegram notification behavior

SkyGuard stores a user's last submitted coordinates and Telegram chat ID. When the backend processes an eligible threat report, it calculates the distance between the report and users who have notifications enabled. Users within **30 km** may receive a Telegram message.

To protect account linking, the application generates a one-time code that expires after **10 minutes**. A user opens the generated bot deep link to associate their Telegram chat with their SkyGuard account.

Telegram alerts depend on valid coordinates, a linked chat, a working bot token, and Telegram API availability. Delivery is not guaranteed.

## Security and deployment notes

This repository is a hackathon project and should be reviewed before production deployment.

- Set `DEBUG=False` in production and use a strong, private `SECRET_KEY`.
- Configure `ALLOWED_HOSTS` and CORS origins for the actual deployment domains; do not use wildcard host access.
- Keep `.env`, database files, credentials, and bot tokens out of version control.
- Use HTTPS for the frontend, API, and Telegram webhook endpoint.
- Use a production-ready WSGI/ASGI server and a managed PostgreSQL instance.
- Configure backups and a retention policy for location and report data.
- Apply rate limiting, input validation, and abuse prevention to public-facing endpoints.
- Review location-sharing and notification consent flows before collecting or processing user coordinates.
- Treat community reports as unverified information until reviewed through an authoritative process.

## Project status

SkyGuard is an experimental hackathon prototype. Features and deployment instructions may change as the project evolves. Some components require environment-specific configuration, and the repository should not be considered a ready-to-deploy emergency response system.

## Contributing

Contributions, bug reports, and suggestions are welcome. For substantial changes, open an issue first to discuss the proposed approach. Keep pull requests focused and include clear steps to test the change.

## License

No license file is currently included in this repository. Unless a license is added, the source code should be treated as copyrighted by its respective rights holder; do not assume that it is available for unrestricted reuse.
