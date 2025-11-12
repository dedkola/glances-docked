# Glances Server Docker

Minimal Alpine Linux-based Docker image running Glances in server mode for comprehensive system monitoring.

## About This Project

This project provides a lightweight, production-ready Docker container for [Glances](https://github.com/nicolargo/glances) - a cross-platform system monitoring tool. The container runs Glances in server mode with a web interface, allowing you to monitor system resources (CPU, memory, disk, network, processes) from anywhere.

**Key Features:**
- 🐧 Minimal Alpine Linux base (147MB total image size)
- 📊 Real-time monitoring of CPU, memory, disk, network, processes
- 🌐 Modern web UI with responsive design
- 🔌 RESTful API for programmatic access
- 🖥️ Client mode support for remote monitoring
- 🔒 Lightweight and secure

## Quick Start

### Using Docker Compose (Recommended)
```bash
docker-compose up -d
```

### Using Docker CLI
```bash
docker run -d \
  --name glances \
  -p 61208:61208 \
  --pid=host \
  --restart=unless-stopped \
  glances-server
```

## Step-by-Step Guide

### 1. Build the Docker Image

```bash
# Clone or navigate to the project directory
cd /path/to/glances

# Build the image
docker build -t glances-server .

# Verify the build
docker images glances-server
```

### 2. Run the Container

**Option A: Using Docker Compose**
```bash
# Start the container
docker-compose up -d

# View logs
docker-compose logs -f

# Stop the container
docker-compose down
```

**Option B: Using Docker CLI**
```bash
# Run the container
docker run -d \
  --name glances \
  -p 61208:61208 \
  --pid=host \
  --restart=unless-stopped \
  glances-server

# View logs
docker logs -f glances

# Stop the container
docker stop glances
docker rm glances
```

### 3. Tag the Image

```bash
# Tag for Docker Hub (replace 'yourusername' with your Docker Hub username)
docker tag glances-server yourusername/glances-server:latest
docker tag glances-server yourusername/glances-server:v1.0.0

# Verify tags
docker images | grep glances-server
```

### 4. Deploy to Docker Hub

```bash
# Login to Docker Hub
docker login

# Push the image
docker push yourusername/glances-server:latest
docker push yourusername/glances-server:v1.0.0
```

### 5. Pull and Run from Docker Hub

```bash
# Pull the image (on any machine)
docker pull yourusername/glances-server:latest

# Run the container
docker run -d \
  --name glances \
  -p 61208:61208 \
  --pid=host \
  --restart=unless-stopped \
  yourusername/glances-server:latest
```

## Access Methods

### Web Interface
Open your browser and navigate to:
- **Local**: http://localhost:61208
- **Remote**: http://YOUR_SERVER_IP:61208

### RESTful API
Access system metrics programmatically:
```bash
# Get API version
curl http://localhost:61208/api/4/status

# Get CPU info
curl http://localhost:61208/api/4/cpu

# Get memory info
curl http://localhost:61208/api/4/mem

# Get all stats
curl http://localhost:61208/api/4/all
```

### Glances Client Mode
Connect from another machine using the Glances client:
```bash
# Install glances on client machine
pip install glances

# Connect to the server
glances -c YOUR_SERVER_IP -p 61208
```

## Configuration

### Environment Variables
Customize the container using environment variables in `docker-compose.yml`:

```yaml
environment:
  - TZ=America/New_York          # Set timezone
  - GLANCES_OPT=-w               # Glances options
```

### Ports
- `61208`: Web interface and API (default Glances web port)
- `61209`: Alternative client connection port (if needed)

### Volumes
To persist configuration (optional):
```yaml
volumes:
  - ./glances.conf:/etc/glances/glances.conf:ro
```

## Docker Image Details

**Base Image**: Alpine Linux (latest)
**Size**: ~147MB
**Installed Packages**:
- Python 3.12
- Glances (latest) with web dependencies
- FastAPI & Uvicorn (for web server)
- psutil (for system monitoring)

## Security Considerations

- The container runs with `--pid=host` to access host processes
- No privileged mode required for basic monitoring
- Bind to `0.0.0.0` to allow external connections (use firewall rules for production)
- Consider adding authentication for production use

## Troubleshooting

### Container won't start
```bash
# Check logs
docker logs glances

# Check if port is already in use
netstat -tulpn | grep 61208
```

### Can't access web interface
```bash
# Verify container is running
docker ps | grep glances

# Check port mapping
docker port glances

# Test locally
curl http://localhost:61208
```

### No system metrics showing
- Ensure `--pid=host` flag is set
- Check container logs for errors

## Advanced Usage

### Custom Glances Configuration
Create a `glances.conf` file and mount it:
```bash
docker run -d \
  --name glances \
  -p 61208:61208 \
  --pid=host \
  -v $(pwd)/glances.conf:/etc/glances/glances.conf:ro \
  glances-server
```

### Monitor Docker Containers
Mount Docker socket to monitor containers:
```bash
docker run -d \
  --name glances \
  -p 61208:61208 \
  --pid=host \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  glances-server
```

## Project Structure

```
.
├── Dockerfile              # Docker image definition
├── docker-compose.yml      # Docker Compose configuration
├── .dockerignore          # Docker build exclusions
└── README.md              # This file
```

## Contributing

Feel free to submit issues, fork the repository, and create pull requests for any improvements.

## License

This project is open source. Glances itself is licensed under LGPL-3.0.

## Links

- **Glances Official**: https://github.com/nicolargo/glances
- **Glances Documentation**: https://glances.readthedocs.io/
- **Docker Hub**: https://hub.docker.com/r/yourusername/glances-server

## Changelog

### v1.0.0 (2025-11-12)
- Initial release
- Alpine Linux base
- Glances with web UI
- FastAPI integration
- Docker Compose support
