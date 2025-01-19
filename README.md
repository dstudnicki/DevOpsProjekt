# Dokumentacja aplikacji Flask i procesu CI/CD

## 1. Opis aplikacji

### Plik `main.py`
Aplikacja oparta na frameworku Flask, obsługująca jeden endpoint (`/`), który wyświetla wiadomość na stronie głównej.

#### Kod aplikacji:
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

### Funkcjonalności:
- **Endpoint `/`**:
  - Renderuje szablon `index.html`.
  - Przekazuje dane (`data`) z wiadomością powitalną.
- Aplikacja uruchamia serwer HTTP na porcie `5000`, nasłuchując na wszystkich interfejsach (`host='0.0.0.0'`).
- Tryb debugowania jest włączony (`debug=True`), co pozwala na szybkie diagnozowanie błędów.

## 2. Plik CI/CD: `ci.yml`

### Cel:
Automatyzacja procesu budowy, testowania, skanowania bezpieczeństwa oraz publikowania obrazu Dockera do GitHub Container Registry (GHCR).

### Struktura pliku:

#### Sekcja `on`
```yaml
name: CI workflow

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
```
- Workflow uruchamia się automatycznie przy każdym `push` na branch `main`.
- Workflow uruchamia się przy każdym `pull request` skierowanym do brancha `main`.

#### Sekcja `jobs`
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
```
- Workflow działa na maszynie z systemem `Ubuntu-latest`.

### Kroki (steps):

#### 1. Checkout kodu
```yaml
- name: Checkout code
  uses: actions/checkout@v3
```
Zapewnia pobranie najnowszej wersji kodu źródłowego z repozytorium.

#### 2. Konfiguracja środowiska Python
```yaml
- name: Set up Python
  uses: actions/setup-python@v4
  with:
    python-version: "3.10"
```
Instaluje środowisko Python w wersji `3.10`.

#### 3. Instalacja zależności
```yaml
- name: Install dependencies
  run: |
    python -m pip install --upgrade pip
```
Aktualizuje menedżer pakietów `pip`.

#### 4. Uruchomienie testów
```yaml
- name: Run basic test
  run: echo "Test"
```
Placeholder dla uruchamiania testów aplikacji.

#### 5. Budowanie obrazu Docker
```yaml
- name: Build Docker image
  run: docker build -t flask-app .
```
Budowanie obrazu Dockera na podstawie pliku `Dockerfile`.

#### 6. Skanowanie bezpieczeństwa (Trivy)
```yaml
- name: Run Trivy scan
  run: |
    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image flask-app
```
Skanowanie obrazu Docker pod kątem podatności z użyciem narzędzia **Trivy**.

#### 7. Publikacja do Github Container Registry
```yaml
- name: Push to GitHub Container Registry
  if: github.event_name == 'push'
  run: |
    docker login --username dstudnicki --password ${{ secrets.GH_PAT }} ghcr.io
    docker build . --tag ghcr.io/dstudnicki/flask-app:latest
    docker push ghcr.io/dstudnicki/flask-app:latest
```
- Logowanie do GHCR z użyciem **Personal Access Token (GH_PAT)**.
- Budowanie obrazu Dockera i tagowanie go jako `ghcr.io/dstudnicki/flask-app:latest`.
- Wypchnięcie obrazu do GitHub Container Registry.

## 3. Proces uruchamiania aplikacji

### 1. Uruchomienie lokalne:
#### Budowanie obrazu Docker aplikacji:
```sh
docker build -t flask-app .
```
#### Uruchomienie obrazu oraz aplikacji na porcie `5000`:
```sh
docker run -p 5000:5000 flask-app
```
Aplikacja będzie dostępna pod adresem: **http://localhost:5000**.

### 2. Uruchomienie w Dockerze z GHCR:
#### Pobierz obraz Docker z GHCR:
```sh
docker pull ghcr.io/dstudnicki/flask-app:latest
```
#### Uruchom obraz Docker:
```sh
docker run -p 5000:5000 ghcr.io/dstudnicki/flask-app:latest
```
Aplikacja będzie dostępna pod adresem: **http://localhost:5000**.

---

