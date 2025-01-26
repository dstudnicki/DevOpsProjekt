# Flask Application and CI/CD Documentation

## 1. Application Overview

### `main.py`

The application is based on the Flask framework, handling a single endpoint (`/`) that displays a message on the home page.

#### Application Code:

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def home():
    data = {"message": "Welcome to my Flask app!"}
    return render_template('index.html', data=data)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

### Features:

- **Endpoint **``:
  - Renders the `index.html` template.
  - Passes data (`data`) containing a welcome message.
- The application runs an HTTP server on port `5000`, listening on all interfaces (`host='0.0.0.0'`).
- Debug mode is enabled (`debug=True`) for easier error diagnostics.

---

## 2. CI/CD Pipeline: `ci.yml`

### Goal:

Automate the process of building, testing, performing security scans, and publishing the Docker image to GitHub Container Registry (GHCR).

### Workflow Structure:

#### `on` Section:

```yaml
name: CI workflow

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
```

- The workflow triggers automatically on any `push` to the `main` branch.
- It also triggers for pull requests targeting the `main` branch.

#### `jobs` Section:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
```

- The workflow runs on an `Ubuntu-latest` machine.

### Steps:

#### 1. Checkout Code:

```yaml
- name: Checkout code
  uses: actions/checkout@v3
```

Fetches the latest version of the source code from the repository.

#### 2. Set Up Python:

```yaml
- name: Set up Python
  uses: actions/setup-python@v4
  with:
    python-version: "3.10"
```

Installs Python version `3.10`.

#### 3. Install Dependencies:

```yaml
- name: Install dependencies
  run: |
    python -m pip install --upgrade pip
```

Upgrades the `pip` package manager.

#### 4. Run Tests:

```yaml
- name: Run basic test
  run: echo "Test"
```

Placeholder for running application tests.

#### 5. Build Docker Image:

```yaml
- name: Build Docker image
  run: docker build -t flask-app .
```

Builds the Docker image using the `Dockerfile`.

#### 6. Security Scan with Trivy:

```yaml
- name: Run Trivy scan
  run: |
    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image flask-app
```

Scans the Docker image for vulnerabilities using **Trivy**.

#### 7. Publish to GitHub Container Registry:

```yaml
- name: Push to GitHub Container Registry
  if: github.event_name == 'push'
  run: |
    docker login --username dstudnicki --password ${{ secrets.GH_PAT }} ghcr.io
    docker build . --tag ghcr.io/dstudnicki/flask-app:latest
    docker push ghcr.io/dstudnicki/flask-app:latest
```

- Logs in to GHCR using a **Personal Access Token (GH\_PAT)**.
- Builds the Docker image and tags it as `ghcr.io/dstudnicki/flask-app:latest`.
- Pushes the image to GitHub Container Registry.

---

## 3. Running the Application

### 1. Local Execution:

#### Build the Docker Image:

```sh
docker build -t flask-app .
```

#### Run the Docker Container:

```sh
docker run -p 5000:5000 flask-app
```

The application will be available at: [**http://localhost:5000**](http://localhost:5000).

### 2. Running with GHCR:

#### Pull the Docker Image from GHCR:

```sh
docker pull ghcr.io/dstudnicki/flask-app:latest
```

#### Run the Docker Container:

```sh
docker run -p 5000:5000 ghcr.io/dstudnicki/flask-app:latest
```

The application will be available at: [**http://localhost:5000**](http://localhost:5000).

