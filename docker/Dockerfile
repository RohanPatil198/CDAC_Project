# docker/Dockerfile
# Multi-stage build
FROM python:3.11-slim AS build
WORKDIR /app
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1

# Install build deps
RUN apt-get update && apt-get install -y --no-install-recommends build-essential gcc libmariadb-dev-compat libmariadb-dev && \
    rm -rf /var/lib/apt/lists/*

COPY app/requirements.txt /app/requirements.txt
RUN pip install --upgrade pip && pip wheel --wheel-dir=/wheels -r requirements.txt

FROM python:3.11-slim
WORKDIR /app
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1

# runtime deps for mysql connector etc
RUN apt-get update && apt-get install -y --no-install-recommends libmariadb3 libpq5 && \
    rm -rf /var/lib/apt/lists/*

# copy wheels
COPY --from=build /wheels /wheels
RUN pip install --no-cache /wheels/*

# app code
COPY app /app

# create non-root user
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 5000
ENV FLASK_ENV=production
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app", "--workers", "3", "--threads", "2", "--timeout", "120"]
