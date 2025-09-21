# Redis Insight on macOS

Redis Insight is a powerful tool to visualize, monitor, and optimize Redis instances.  
This guide explains how to install and run Redis Insight on macOS using Docker.

---

## Prerequisites

- **Homebrew**: Ensure you have [Homebrew](https://brew.sh/) installed on your macOS.
- **Docker Desktop**: Redis Insight runs in Docker, so Docker Desktop is required.

---

## Installation Steps

### 1. Install Docker
```bash
brew install --cask docker
````

### 2. Start Docker

```bash
open /Applications/Docker.app
```

Make sure Docker is running before proceeding.

### 3. Run Redis Insight with Docker

```bash
docker run -d \
  --name redisinsight \
  -p 5540:5540 \
  redislabs/redisinsight:latest
```

---

## Access Redis Insight

Once the container is running, open your browser and navigate to:

👉 [http://localhost:5540](http://localhost:5540)

---

## Container Management

* **Stop Redis Insight**

  ```bash
  docker stop redisinsight
  ```

* **Start Redis Insight again**

  ```bash
  docker start redisinsight
  ```

* **Remove Redis Insight**

  ```bash
  docker rm -f redisinsight
  ```
