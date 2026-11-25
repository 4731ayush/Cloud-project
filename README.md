# Cloud-project
1. app.py
from flask import Flask, jsonify
import os
import redis

app = Flask(__name__)

redis_client = redis.from_url(
    os.getenv("REDIS_URL", "redis://redis:6379/0"),
    decode_responses=True
)

@app.route("/")
def visitor_counter():
    count = redis_client.incr("visitors")
    return f"""
    <html>
      <body style="font-family:Arial;text-align:center;margin-top:100px">
        <h1>Kubernetes Visitor Counter</h1>
        <h2>{count}</h2>
        <p>Visitors served using Flask and Redis</p>
      </body>
    </html>
    """

@app.route("/health")
def health():
    redis_client.ping()
    return jsonify(status="ok", redis="ready")

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
2. requirements.txt
Flask==3.0.3
redis==5.0.7
gunicorn==22.0.0
3. Dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "2", "app:app"]
4. docker-compose.yml
services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      REDIS_URL: redis://redis:6379/0
    depends_on:
      redis:
        condition: service_healthy

  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
