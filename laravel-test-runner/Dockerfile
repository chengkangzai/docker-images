FROM kirschbaumdevelopment/laravel-test-runner:8.4

# pcov backs Pest's test impact analysis (--tia): without a coverage driver Pest
# skips TIA silently and runs the whole suite. Baked in rather than apt-installed
# per job so CI does not depend on a Debian mirror on every shard.
RUN apt-get update \
    && apt-get install -y --no-install-recommends php8.4-imagick php8.4-pcov ffmpeg \
    && rm -rf /var/lib/apt/lists/*

# CVE-2026-45793 / GHSA-f9f8-rm49-7jv2: Composer <2.9.8 leaks GitHub Actions
# tokens to CI logs. Upgrade above the Kirschbaum base (2.9.5).
RUN composer self-update --2 --no-interaction
