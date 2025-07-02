# **Building Spokane Tech: Part 9**

Welcome to part 9 of the "Building Spokane Tech" series! In this article, we'll optimize our Docker file for size and efficiency.

> See the live site at: [https://www.spokanetech.org](https://www.spokanetech.org)

> See the latest code on: [github](https://github.com/SpokaneTech/SpokaneTechWeb)


## **Optimizing Your Django Docker Images: The Power of Multi-Stage Builds**
At SpokaneTech.org, we're always looking for ways to improve our development and deployment processes. A cornerstone of modern web application deployment is containerization, and for that, we rely on Docker. Docker allows us to package our Django application with all of its dependencies into a single, portable container.

However, as applications grow, so can the complexity and size of their Docker images. A bloated Docker image can lead to slower deployment times, increased storage costs, and a larger attack surface. Today, we'll explore how we optimized our Docker workflow for our Django project by transitioning from a single-stage to a multi-stage Dockerfile.

## **The Starting Point: A Single-Stage Dockerfile**
When you first start Dockerizing a project, you'll likely begin with a single-stage Dockerfile. It's straightforward and gets the job done. Here’s what our initial Dockerfile looked like:

```
FROM python:3.12-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Set the working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    libglib2.0-0 \
    libnss3 \
    libx11-xcb1 \
    libxcomposite1 \
    libxrandr2 \
    libxdamage1 \
    libgbm1 \
    libasound2 \
    libpangocairo-1.0-0 \
    netcat-openbsd \
    && rm -rf /var/lib/apt/lists/*

COPY pyproject.toml /app/
COPY src/django_project /app/

RUN chmod +x /app/entrypoint.sh
RUN pip install --upgrade pip pip-tools
RUN pip install .[docker]
RUN playwright install --with-deps

# Expose the port that the app runs on
EXPOSE 8000

CMD ["gunicorn", "core.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "3"]

```

This Dockerfile works, but it has some significant drawbacks. It installs all our Python dependencies, including development and testing libraries like pip-tools and playwright, directly into the final image. It also includes the system dependencies required to install Playwright's browsers, not just run them. The result is a larger-than-necessary image that contains tools we don't need in a production environment.



## **The Solution: Embracing Multi-Stage Builds**
A multi-stage build is a powerful feature in Docker that allows you to use multiple FROM statements in a single Dockerfile. Each FROM instruction begins a new "stage" of the build. You can selectively copy artifacts—like compiled code or installed packages—from one stage to another, leaving behind everything you don't need.

Let's look at our new and improved multi-stage Dockerfile:

```
# ====================
# Stage 1: Builder
# ====================
FROM python:3.12-slim AS builder

WORKDIR /app

# Install system dependencies needed for building and running Chromium
RUN apt-get update && apt-get install -y --no-install-recommends \
    libglib2.0-0 libnss3 libx11-xcb1 libxcomposite1 libxrandr2 \
    libxdamage1 libgbm1 libasound2 libpangocairo-1.0-0 \
    libatk-bridge2.0-0 libgtk-3-0 fonts-liberation libxshmfence1 \
    libxcb1 xdg-utils netcat-openbsd \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install uv (fast dependency resolver) and create virtualenv
RUN pip install --no-cache-dir uv \
    && python -m venv /venv

ENV PATH="/venv/bin:$PATH"

# Copy project dependency definition
COPY pyproject.toml /app/

# Install Python dependencies (including Playwright and its deps)
RUN uv pip install .[docker] --prerelease=allow \
    && playwright install chromium --with-deps

# ====================
# Stage 2: Runtime
# ====================
FROM python:3.12-slim

WORKDIR /app

# Install only necessary system dependencies for Chromium to run
RUN apt-get update && apt-get install -y --no-install-recommends \
    libglib2.0-0 libnss3 libx11-xcb1 libxcomposite1 libxrandr2 \
    libxdamage1 libgbm1 libasound2 libpangocairo-1.0-0 \
    libatk-bridge2.0-0 libgtk-3-0 fonts-liberation libxshmfence1 \
    libxcb1 xdg-utils netcat-openbsd \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Environment settings
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PATH="/venv/bin:$PATH"
ENV PLAYWRIGHT_BROWSERS_PATH=/root/.cache/ms-playwright

# Copy the virtual environment and browser binaries from the builder
COPY --from=builder /venv /venv
COPY --from=builder /root/.cache/ms-playwright /root/.cache/ms-playwright

# Copy Django app code
COPY src/django_project /app/

# Make sure entrypoint script is executable
RUN chmod +x /app/entrypoint.sh

# Expose app port
EXPOSE 8000

# Entrypoint handles DB wait, migrations, and server start
CMD ["./entrypoint.sh"]

```

This Dockerfile is split into two distinct stages:

1. The builder Stage: This first stage is where the heavy lifting happens. We install all the necessary build tools and system libraries. We use the speedy uv installer to create a virtual environment and install all our Python dependencies. We also download and set up the Playwright browser binaries. This stage is a complete development and build environment.

2. The runtime Stage: This is the stage that will become our final production image. It starts from a fresh, clean python:3.12-slim image. Notice what we copy from the builder stage using the --from=builder flag:

    - The entire Python virtual environment (/venv).

    - The installed Playwright browser (/root/.cache/ms-playwright).

We then copy in our application code. The final image contains only our Python environment, browser binaries, our code, and the minimal system dependencies needed to run them—nothing else.


## **The Key Benefits of Going Multi-Stage**
So, what have we gained from this approach?

- Dramatically Smaller Image Sizes: By excluding build tools, -dev packages, and other intermediate files, the final image is significantly smaller. This means faster push/pull times from our container registry, quicker deployments, and reduced storage costs.

- Enhanced Security: A smaller image has a smaller attack surface. By not shipping our build tools, compilers, or development dependencies, we remove potential vulnerabilities that could be exploited in a production environment.

- Improved Caching and Faster Builds: Docker is smart about caching layers. In our multi-stage setup, if we only change our application code (src/django_project), Docker can use the cached builder stage without re-installing all the dependencies, speeding up subsequent builds.

- Cleaner and More Maintainable Dockerfiles: Separating the build and runtime concerns makes the Dockerfile more organized and easier to understand. It clearly communicates what is needed to build the application versus what is needed to run it.


## **Conclusion**
While a single-stage Dockerfile is a great way to get started, adopting a multi-stage build process is a crucial step toward creating professional, production-ready container images. For SpokaneTech.org, this switch has resulted in a more efficient, secure, and streamlined deployment pipeline for our Django application. If you're not already using multi-stage builds, we highly recommend giving them a try!
