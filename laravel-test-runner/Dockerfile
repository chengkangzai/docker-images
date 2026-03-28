FROM kirschbaumdevelopment/laravel-test-runner:8.4

RUN apt-get update \
    && apt-get install -y --no-install-recommends php8.4-imagick ffmpeg \
    && rm -rf /var/lib/apt/lists/*
