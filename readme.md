# Dockerized Flask Application

A lightweight Flask web application containerized using Docker.

## How to Run

1. **Build the Docker Image:**
   ```bash
   docker build -t myflaskapp .
   ```

2. **Run the Container:**
   ```bash
   docker run -p 5000:5000 myflaskapp
   ```

3. **Access the App:**
   Open `http://localhost:5000` in your browser.
