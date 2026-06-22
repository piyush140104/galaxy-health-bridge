# syntax=docker/dockerfile:1.7
FROM python:3.14-slim-bookworm AS base
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends \
      build-essential libpq-dev ca-certificates && \
    rm -rf /var/lib/apt/lists/*

COPY pyproject.toml ./
RUN pip install --upgrade pip && pip install .

COPY . .

RUN groupadd -r worker && useradd -r -g worker -d /app -s /sbin/nologin worker
USER worker

# Beat + worker in one container for simplicity. Split via compose if needed.
CMD ["celery","-A","app.main:celery","worker","--loglevel=INFO","-Q","default,sync,retention","--concurrency=4"]
