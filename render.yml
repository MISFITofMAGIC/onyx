databases:
  - name: onyx-db
    engine: postgres
    version: 15
    plan: standard
    region: oregon
    replicas: 1

services:
  - type: web
    name: onyx-api-server
    env: docker
    plan: standard
    region: oregon
    dockerfilePath: backend/Dockerfile
    buildCommand: |
      alembic upgrade head &&
      echo "Starting Onyx API Server"
    startCommand: |
      uvicorn onyx.main:app --host 0.0.0.0 --port 8080
    envVars:
      - key: AUTH_TYPE
        value: disabled
      - key: POSTGRES_HOST
        value: onyx-db
      - key: VESPA_HOST
        value: vespa-index
      - key: REDIS_HOST
        value: onyx-cache
      - key: MODEL_SERVER_HOST
        value: inference-model-server
      - key: USE_IAM_AUTH
        value: false
      - key: DATABASE_URL
        fromDatabase:
          name: onyx-db
          property: connectionString

  - type: web
    name: onyx-web-server
    env: docker
    plan: standard
    region: oregon
    dockerfilePath: web/Dockerfile
    buildCommand: |
      echo "Building Onyx Web Server"
    startCommand: |
      npm start
    envVars:
      - key: INTERNAL_URL
        value: http://onyx-api-server:8080

  - type: worker
    name: onyx-background-worker
    env: docker
    plan: standard
    region: oregon
    dockerfilePath: backend/Dockerfile
    buildCommand: |
      echo "Starting Onyx Background Worker"
    startCommand: |
      /usr/bin/supervisord -c /etc/supervisor/conf.d/supervisord.conf
    envVars:
      - key: AUTH_TYPE
        value: disabled
      - key: POSTGRES_HOST
        value: onyx-db
      - key: VESPA_HOST
        value: vespa-index
      - key: REDIS_HOST
        value: onyx-cache
      - key: MODEL_SERVER_HOST
        value: inference-model-server
      - key: INDEXING_MODEL_SERVER_HOST
        value: indexing-model-server
      - key: USE_IAM_AUTH
        value: false
      - key: DATABASE_URL
        fromDatabase:
          name: onyx-db
          property: connectionString

  - type: private
    name: inference-model-server
    env: docker
    plan: standard
    region: oregon
    dockerfilePath: backend/Dockerfile.model_server
    buildCommand: |
      echo "Building Inference Model Server"
    startCommand: |
      uvicorn model_server.main:app --host 0.0.0.0 --port 9000
    envVars:
      - key: MIN_THREADS_ML_MODELS
        value: 1
      - key: LOG_LEVEL
        value: info

  - type: private
    name: indexing-model-server
    env: docker
    plan: standard
    region: oregon
    dockerfilePath: backend/Dockerfile.model_server
    buildCommand: |
      echo "Building Indexing Model Server"
    startCommand: |
      uvicorn model_server.main:app --host 0.0.0.0 --port 9000
    envVars:
      - key: MIN_THREADS_ML_MODELS
        value: 1
      - key: INDEXING_ONLY
        value: true
      - key: LOG_LEVEL
        value: info
      - key: VESPA_SEARCHER_THREADS
        value: 1

  - type: private
    name: vespa-index
    env: docker
    plan: standard
    region: oregon
    dockerImage: vespaengine/vespa:8.277.17
    buildCommand: |
      echo "Starting Vespa Index"
    startCommand: |
      vespa-start
    envVars:
      - key: VESPA_PORT
        value: 8081

  - type: private
    name: onyx-cache
    env: docker
    plan: standard
    region: oregon
    dockerImage: redis:7.4-alpine
    buildCommand: |
      echo "Starting Redis Cache"
    startCommand: |
      redis-server --save "" --appendonly no
    envVars:
      - key: REDIS_PORT
        value: 6379
