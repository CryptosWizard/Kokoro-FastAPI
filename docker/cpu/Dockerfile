FROM python:3.10-slim 

# Install dependencies and check espeak location
RUN apt-get update && apt-get install -y \
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

# Set the working directory
WORKDIR /app

# Copy all files as root (default behavior without --chown)
COPY pyproject.toml ./pyproject.toml
COPY api ./api
COPY web ./web
COPY docker/scripts/ ./scripts/

# Make the entrypoint script executable
RUN chmod +x ./scripts/entrypoint.sh

# Change ownership of all files in /app to appuser:appuser
RUN chown -R appuser:appuser /app

# Switch to appuser for subsequent RUN, CMD, and ENTRYPOINT instructions
USER appuser

# Install dependencies using uv with a cache mount accessible to appuser
RUN --mount=type=cache,target=/app/.cache/uv \
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

# Run the FastAPI server through the entrypoint script
CMD ["./scripts/entrypoint.sh"]
