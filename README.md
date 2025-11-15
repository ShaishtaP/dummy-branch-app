# Branch Loan API (Python/Flask)

This repository contains the solution for the DevOps Intern Take-Home Assignment. It implements a production-ready, containerized Loan API (using Python/Flask) with a multi-environment Docker Compose setup, CI/CD pipeline, and comprehensive documentation.

## 🏛 Architecture Diagram

Here is a simple diagram of the local multi-container setup:
##  How to Run Locally (Step-by-Step)

1.  *Clone the Repository*
    bash
    git clone [your-fork-url]
    cd [repository-name]
    

2.  *Edit Your Hosts File*
    You must map branchloans.com to your local machine to satisfy the HTTPS requirement.
    * *Mac/Linux:* sudo nano /etc/hosts
    * *Windows (as Admin):* notepad C:\Windows\System32\drivers\etc\hosts

    Add this line to the file and save:
    
    127.0.0.1   branchloans.com
    

3.  *Generate SSL Certificate*
    This one-time command creates the nginx/certs folder and a self-signed certificate for Nginx to use.
    bash
    mkdir -p nginx/certs
    openssl req -x509 -newkey rsa:2048 -nodes \
      -keyout nginx/certs/key.pem \
      -out nginx/certs/cert.pem \
      -days 365 \
      -subj "//C=US/ST=California/L=SanFrancisco/O=Branch/CN=branchloans.com"
    
    *(Note: The // at the start of the subject is required for Git Bash on Windows.)*

4.  *Run the Development Environment*
    This will build the images, start the containers, and mount your local code for hot-reloading.
    bash
    # Copy the dev environment file
    cp .env.dev .env
    
    # Run docker compose with the dev override
    docker-compose -f docker-compose.yml -f docker-compose.dev.yml up --build
    

5.  *Access the API*
    Open your browser and go to https://branchloans.com/health.
    * You will see a security warning. Click "Advanced" and "Proceed."
    * You should see: {"status":"healthy","database":"connected"}

##  How to Switch Environments

This setup supports dev, staging, and prod environments using override files. To switch, stop Docker Compose (Ctrl+C), copy the correct .env file, and run the appropriate command:

* *Development:* (Hot-reload enabled via Flask's debug server)
    bash
    cp .env.dev .env
    docker-compose -f docker-compose.yml -f docker-compose.dev.yml up
    

* *Staging:* (Mimics prod with resource limits)
    bash
    cp .env.staging .env
    docker-compose -f docker-compose.yml -f docker-compose.staging.yml up
    

* *Production:* (Persistent data, healthchecks, and Gunicorn server)
    bash
    cp .env.prod .env
    docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
    

##  Environment Variables

These variables are defined in the .env.* files and control the application's configuration.

| Variable | Description | Example (dev) |
| :--- | :--- | :--- |
| PORT | Port the API server listens on inside the container. | 8080 |
| LOG_LEVEL | Sets the logging verbosity. json for prod. | debug |
| POSTGRES_USER | Username for the PostgreSQL database. | dev_user |
| POSTGRES_PASSWORD | Password for the PostgreSQL database. | dev_secret |
| POSTGRES_DB | Name of the PostgreSQL database. | loans_dev |
| DB_HOST | Hostname for the database (set to the service name db).| db |

##  How the CI/CD Pipeline Works

The CI/CD pipeline is defined in .github/workflows/ci-cd.yml and runs on every push or pull request to the main branch.

1.  *Test Stage:*
    * Sets up a live PostgreSQL database as a "service container".
    * Sets up Python and manually installs all Python packages (e.g., pip install Flask...). This is a special fix to bypass requirements.txt issues.
    * Runs the pytest suite against the live test database. The pipeline stops if tests fail.

2.  *Build Stage:*
    * Runs only if the test stage passes.
    * Builds the Docker image from the Dockerfile.
    * Tags the image with the Git commit SHA.

3.  *Security Scan Stage:*
    * Scans the built Docker image for CRITICAL or HIGH vulnerabilities using Trivy.
    * The pipeline fails if critical vulnerabilities are found.

4.  *Push Stage:*
    * Runs only if all previous stages pass *and* the trigger was a push to the main branch (not a PR).
    * Logs in to GitHub Container Registry (ghcr.io).
    * Pushes the image to the registry with the commit SHA and latest tags.

## ⚠ Troubleshooting

* *How to check if everything is running?*
    1.  Run docker ps. You should see three containers running: api, db, and proxy.
    2.  Run docker-compose logs -f api to see the live logs from the Python application.
    3.  Check the health endpoint: https://branchloans.com/health.

