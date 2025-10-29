# ----------------------------------------
# STAGE 1: Build Dependencies (Builder)
# ----------------------------------------
FROM python:3.10-slim-bullseye AS builder

# Set environment variable for non-interactive apt-get
ENV DEBIAN_FRONTEND=noninteractive

# FIX: Completely replace sources.list with the official, standard, and reliable
# CDN mirrors (deb.debian.org for main/updates and security.debian.org for security).
# This prevents conflicts and hardcoded mirror issues.
RUN echo "deb http://deb.debian.org/debian bullseye main contrib non-free" > /etc/apt/sources.list \
    && echo "deb http://deb.debian.org/debian bullseye-updates main contrib non-free" >> /etc/apt/sources.list \
    && echo "deb http://security.debian.org/debian-security bullseye-security main contrib non-free" >> /etc/apt/sources.list

# Install system dependencies
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        build-essential \
        libpq-dev \
        python3-dev \
        ffmpeg \
    && rm -rf /var/lib/apt/lists/*

# Set the working directory
WORKDIR /app

# Copy only the requirements file to leverage Docker layer caching
COPY ./requirements.txt /app/

# Install the Python dependencies
RUN pip install --no-cache-dir --upgrade pip \
    && pip install --no-cache-dir --requirement /app/requirements.txt


# ----------------------------------------
# STAGE 2: Final Runtime Image
# ----------------------------------------
FROM python:3.10-slim-bullseye

# Set environment variable for non-interactive apt-get
ENV DEBIAN_FRONTEND=noninteractive
WORKDIR /app

# FIX: Set the clean sources.list again for the final image before installing runtime libs
RUN echo "deb http://deb.debian.org/debian bullseye main contrib non-free" > /etc/apt/sources.list \
    && echo "deb http://deb.debian.org/debian bullseye-updates main contrib non-free" >> /etc/apt/sources.list \
    && echo "deb http://security.debian.org/debian-security bullseye-security main contrib non-free" >> /etc/apt/sources.list

# Copy the system libraries needed at runtime (libpq5, ffmpeg)
# We need to run apt-get update again to use the new sources.list
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        libpq5 \
        ffmpeg \
    && rm -rf /var/lib/apt/lists/*

# Copy the installed Python packages from the builder stage
COPY --from=builder /usr/local/lib/python3.10/site-packages /usr/local/lib/python3.10/site-packages

# Copy the rest of the application code
COPY . /app

# Define the default command to run the service
CMD ["python", "consumer.py"]
