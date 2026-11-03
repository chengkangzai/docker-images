# One Dockerfile for every PHP version: the build workflow passes PHP_VERSION from its
# matrix and tags the image with it, so 8.4 and 8.5 stay identical apart from PHP.
ARG PHP_VERSION=8.4
FROM kirschbaumdevelopment/laravel-test-runner:${PHP_VERSION}
ARG PHP_VERSION

# pcov backs Pest's test impact analysis (--tia): without a coverage driver Pest
# skips TIA silently and runs the whole suite. Baked in rather than apt-installed
# per job so CI does not depend on a Debian mirror on every shard.
RUN apt-get update \
    && apt-get install -y --no-install-recommends php${PHP_VERSION}-imagick php${PHP_VERSION}-pcov ffmpeg \
    && rm -rf /var/lib/apt/lists/*

# Chromium's shared-library dependencies for Pest browser tests — the OS packages
# ONLY, never the browser binary.
#
# Playwright pins a browser revision per release, so a baked Chromium would go
# stale the moment a project bumps its playwright dependency, and fail with
# "Executable doesn't exist". The binary therefore stays a per-job
# `npx playwright install chromium`, which always matches the project's lockfile
# and is cached on the runner's PLAYWRIGHT_BROWSERS_PATH volume. The system libs
# have no such coupling: they are ordinary apt packages that change across Ubuntu
# releases, not across Playwright versions.
#
# This has to happen at image build time. A CI job runs in a throwaway container,
# so apt packages installed during the job are discarded when it ends, while the
# browser binary survives on a mounted volume. That asymmetry meant a
# `playwright install --with-deps` in the job re-resolved the same already-correct
# packages on every single run — ~19s each, forever, with no way to cache it.
#
# Pinned so image builds are reproducible. If a future Playwright needs a library
# this list lacks, consumers self-heal: launching Chromium fails, and they fall
# back to `--with-deps` for that run. Bump the pin here to make it fast again.
RUN npx --yes playwright@1.56.1 install-deps chromium \
    && rm -rf /var/lib/apt/lists/*

# CVE-2026-45793 / GHSA-f9f8-rm49-7jv2: Composer <2.9.8 leaks GitHub Actions
# tokens to CI logs. Upgrade above the Kirschbaum base (2.9.5).
RUN composer self-update --2 --no-interaction

# GitHub CLI for jobs that drive GitHub from inside this container — e.g. Pixalink's
# Sentry Fix workflow, where Claude runs `gh issue view/comment`, `gh pr create` and
# `gh run watch`. The base image has none, so a run could finish "successfully" while
# being unable to read or comment on its own issue.
#
# From the pinned release tarball, verified against GitHub's published checksums. Not the
# cli.github.com apt repo: it only serves the latest release, so a version pin there breaks
# the build the day upstream ships. TARGETARCH comes from buildx (amd64 / arm64 legs).
ARG GH_CLI_VERSION=2.102.0
ARG TARGETARCH
RUN set -eux; \
    arch="${TARGETARCH:-$(dpkg --print-architecture)}"; \
    tarball="gh_${GH_CLI_VERSION}_linux_${arch}.tar.gz"; \
    cd /tmp; \
    curl -fsSLO "https://github.com/cli/cli/releases/download/v${GH_CLI_VERSION}/${tarball}"; \
    curl -fsSL "https://github.com/cli/cli/releases/download/v${GH_CLI_VERSION}/gh_${GH_CLI_VERSION}_checksums.txt" \
        | grep " ${tarball}$" | sha256sum -c -; \
    tar -xzf "${tarball}"; \
    install -m 0755 "gh_${GH_CLI_VERSION}_linux_${arch}/bin/gh" /usr/local/bin/gh; \
    rm -rf "/tmp/${tarball}" "/tmp/gh_${GH_CLI_VERSION}_linux_${arch}"; \
    gh --version
