FROM python:3.10-slim

# Install dependencies...
RUN apt-get update && apt-get install -y \
    python3.10 \
    python3-venv \
    espeak-ng \
    espeak-ng-data \
    git \
    libsndfile1 \
    curl \
    ffmpeg \
    g++ \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/* \
 && mkdir -p /usr/share/espeak-ng-data \
 && ln -s /usr/lib/*/espeak-ng-data/* /usr/share/espeak-ng-data/

# Install UV using the installer script
RUN curl -LsSf https://astral.sh/uv/install.sh | sh && \
    mv /root/.local/bin/uv /usr/local/bin/ && \
    mv /root/.local/bin/uvx /usr/local/bin/

# Create non-root user and set up directories and permissions
RUN useradd -m -u 1000 appuser && \
    mkdir -p /app/api/src/models/v1_0 && \
    chown -R appuser:appuser /app

# **** COPY and chown steps as root ****
WORKDIR /app

COPY pyproject.toml ./pyproject.toml
RUN chown appuser:appuser ./pyproject.toml

COPY api ./api
COPY web ./web
COPY docker/scripts/ ./
RUN chown -R appuser:appuser ./api ./web ./entrypoint.sh
RUN chmod +x ./entrypoint.sh

# Now switch to non-root user
USER appuser

# Install dependencies
RUN --mount=type=cache,target=/root/.cache/uv \
    uv venv --python 3.10 && \
    uv sync --extra cpu

# Set environment variables
ENV PYTHONUNBUFFERED=1 \
    PYTHONPATH=/app:/app/api \
    PATH="/app/.venv/bin:$PATH" \
    UV_LINK_MODE=copy \
    USE_GPU=false \
    PHONEMIZER_ESPEAK_PATH=/usr/bin \
    PHONEMIZER_ESPEAK_DATA=/usr/share/espeak-ng-data \
    ESPEAK_DATA_PATH=/usr/share/espeak-ng-data

ENV DOWNLOAD_MODEL=true

# Download model if enabled
RUN if [ "$DOWNLOAD_MODEL" = "true" ]; then \
    python download_model.py --output api/src/models/v1_0; \
    fi

ENV DEVICE="cpu"

# Run FastAPI server through entrypoint.sh
CMD ["./entrypoint.sh"]
