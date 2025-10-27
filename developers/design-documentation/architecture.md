# Architecture

## System Architecture

### Architectural Style

OpenAlgo employs a **Monolithic Application Architecture** with a **RESTful API** interface. The core logic, broker interactions, database management, and API endpoints are contained within a single Flask application process.

Key characteristics:

* **Centralized Codebase:** All components reside within the same project structure.
* **Flask Framework:** Utilizes the Flask microframework for web application structure and request handling.
* **Flask-RESTX:** Leverages Flask-RESTX for building structured REST APIs with Swagger documentation.
* **Blueprints/Namespaces:** Organizes API endpoints and application logic into modular blueprints (Flask) and namespaces (Flask-RESTX).
* **SQLAlchemy:** Uses SQLAlchemy as the Object-Relational Mapper (ORM) for database interactions.

### Technology Stack

* **Programming Language:** Python 3
* **Web Framework:** Flask
* **API Framework:** Flask-RESTX
* **Database ORM:** SQLAlchemy
* **Real-time Communication:** Flask-SocketIO
* **Rate Limiting:** Flask-Limiter
* **Cross-Origin Resource Sharing:** Flask-CORS
* **Authentication:** Flask-Login, Flask-Bcrypt, PyJWT (likely for API keys/tokens)
* **Web Server Gateway Interface (WSGI):** Werkzeug (Flask's default), Gunicorn for production.
* **Database:** (Defined by `DATABASE_URL` environment variable, Sqlite3 based on common usage with SQLAlchemy)
* **Environment Management:** python-dotenv
* **Deployment:** Docker (Dockerfile, docker-compose.yaml), AWS Elastic Beanstalk (`.ebextensions`)
* **Frontend (UI Templates):** Jinja2, DaisyUI  with Tailwind CSS (based on `tailwind.config.js`)

### Directory Structure Overview

* `.ebextensions/`: Configuration files for AWS Elastic Beanstalk deployment.
* `blueprints/`: Contains Flask Blueprints, organizing application features and web routes (e.g., `auth`, `dashboard`, `orders`).
* `broker/`: Core logic for interacting with different stock brokers. Contains subdirectories for each supported broker (e.g., `jainampro`).
* `database/`: SQLAlchemy models, database initialization scripts, and data access logic (e.g., `auth_db.py`, `user_db.py`).
* `design/`: Location for this design documentation.
* `docs/`: Likely contains user-facing documentation or generated docs.
* `restx_api/`: Defines the Flask-RESTX API structure, namespaces, and models.
* `static/`: Static assets for the web UI (CSS, JavaScript, images).
* `strategies/`: Implementation of trading strategies.
* `templates/`: Jinja2 HTML templates for the web UI.
* `utils/`: Common utility functions and classes used across the application (e.g., `env_check.py`, `latency_monitor.py`, `plugin_loader.py`).
* `app.py`: Main Flask application entry point, initializes the app, extensions, and blueprints.
* `requirements.txt`: Lists Python package dependencies.
* `Dockerfile`, `docker-compose.yaml`: Configuration for building and running the application with Docker.
* `.env`, `.sample.env`: Environment variable configuration.

### Component Diagram (Mermaid)

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

```mermaid
```
