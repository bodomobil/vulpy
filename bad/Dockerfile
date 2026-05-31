# ==========================================
# STAGE 1: Build & Dependency Installation
# ==========================================
FROM python:3.11-slim AS builder

# Prevent Python from writing .pyc files and enable unbuffered logging
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /bad

# Install system build dependencies if needed (e.g., gcc, libpq for PostgreSQL)
# Cleans up apt cache in the same layer to keep the build image smaller
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Upgrade pip and install dependencies into a local virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt


# ==========================================
# STAGE 2: Final Runtime Environment
# ==========================================
FROM python:3.11-slim AS runner

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PATH="/opt/venv/bin:$PATH"

WORKDIR /bad

# Create a non-root system user and group for security
RUN groupadd -g 10001 appgroup && \
    useradd -u 10001 -g appgroup -m -s /sbin/nologin appuser

# Copy the pre-built virtual environment from the builder stage
COPY --from=builder /opt/venv /opt/venv

# Copy application source code and set correct ownership
COPY --chown=appuser:appgroup . .

# Switch to the secure non-root user
USER appuser

# Document the port the container expects to bind to
EXPOSE 8000

# Use an explicit array/exec form for the entrypoint
CMD ["python", "vulpy.py"]