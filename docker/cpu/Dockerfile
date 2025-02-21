FROM python:3.10-slim

# --------------------------------------------------------------------------------
# Install system-level dependencies
# --------------------------------------------------------------------------------
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

# --------------------------------------------------------------------------------
# Install UV using the installer script
# --------------------------------------------------------------------------------
RUN curl -LsSf https://astral.sh/uv/install.sh | sh \
 && mv /root/.local/bin/uv /usr/local/bin/ \
 && mv /root/.local/bin/uvx /usr/local/bin/

# --------------------------------------------------------------------------------
# Create non-root user and prepare directories (still root)
# --------------------------------------------------------------------------------
RUN useradd -m -u 1000 appuser \
 && mkdir -p /app/api/src/models/v1_0

WORKDIR /app

# --------------------------------------------------------------------------------
# Copy dependency files and install Python dependencies as root
# (This keeps layering efficient—no need to do chown here yet.)
# --------------------------------------------------------------------------------
COPY pyproject.toml pyproject.toml

RUN --mount=type=cache,target=/root/.cache/uv \
    uv venv --python 3.10 && \
    uv sync --extra cpu

# --------------------------------------------------------------------------------
# Copy project files
# --------------------------------------------------------------------------------
COPY api ./api
COPY web ./web
COPY docker/scripts/ ./
RUN chmod +x ./entrypoint.sh

# --------------------------------------------------------------------------------
# Set environment variables
# --------------------------------------------------------------------------------
ENV PYTHONUNBUFFERED=1 \
    PYTHONPATH=/app:/app/api \
    PATH="/app/.venv/bin:$PATH" \
    UV_LINK_MODE=copy \
    USE_GPU=false \
    PHONEMIZER_ESPEAK_PATH=/usr/bin \
    PHONEMIZER_ESPEAK_DATA=/usr/share/espeak-ng-data \
    ESPEAK_DATA_PATH=/usr/share/espeak-ng-data

ENV DOWNLOAD_MODEL=true

# --------------------------------------------------------------------------------
# (Optional) Download model if enabled
# This still happens as root, so we can write to /app.
# --------------------------------------------------------------------------------
RUN if [ "$DOWNLOAD_MODEL" = "true" ]; then \
    python download_model.py --output api/src/models/v1_0; \
    fi

ENV DEVICE="cpu"

# --------------------------------------------------------------------------------
# Change ownership of /app *once* and switch to non-root user
# --------------------------------------------------------------------------------
RUN chown -R appuser:appuser /app
USER appuser

# --------------------------------------------------------------------------------
# Final command
# --------------------------------------------------------------------------------
CMD ["./entrypoint.sh"]
