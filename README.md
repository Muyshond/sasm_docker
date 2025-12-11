# Docker Assignment - Complete Guide

## Prerequisites

- SSH access to your server
- Docker installed: `apt install docker.io`
- Git repository set up

---

## Assignment 1: Watch Docker Tutorial Videos

### Steps:

1. Watch at least 2 Docker tutorial videos on YouTube
2. Recommended videos:
   - [Docker in 100 Seconds - Fireship](https://www.youtube.com/watch?v=Gjnup-PuquQ)
   - [Docker Tutorial for Beginners - TechWorld with Nana](https://www.youtube.com/watch?v=3c-iBn73dDE)
   - [Learn Docker in 7 Easy Steps - Fireship](https://www.youtube.com/watch?v=gAkwW2tuIqE)

3. **Save the URLs** - you'll need them for Assignment 2!

---

## Assignment 2: Create Docker Image with NGINX

### Step 1: Create Project Directory

```bash
mkdir -p ~/docker-assignment
cd ~/docker-assignment
```

### Step 2: Create index.html

```bash
nano index.html
```

Add this content (replace URLs with YOUR watched videos):

```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Videos I Watched</title>
</head>
<body>
    <h1>Docker Tutorial Videos</h1>
    <ol>
        <li><a href="https://www.youtube.com/watch?v=Gjnup-PuquQ">Docker in 100 Seconds - Fireship</a></li>
        <li><a href="https://www.youtube.com/watch?v=3c-iBn73dDE">Docker Tutorial for Beginners - TechWorld with Nana</a></li>
    </ol>
</body>
</html>
```

Save: `Ctrl+X`, then `Y`, then `Enter`

### Step 3: Create Dockerfile

**Note**: The assignment asks for `FROM alpine`, but due to permission issues on restricted servers, use `FROM nginx:alpine` instead (which is still based on Alpine Linux).

```bash
nano Dockerfile
```

Add this content:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Save: `Ctrl+X`, then `Y`, then `Enter`

### Step 4: Build the Docker Image

```bash
docker build -t sasm_docker .
```

Expected output:
```
Successfully built [IMAGE_ID]
Successfully tagged sasm_docker:latest
```

### Step 5: Test Locally (Optional)

**Note**: May fail on restricted servers due to permission issues. If it fails, skip to Assignment 3.

```bash
# Try to run
docker run -d -p 8080:80 --name test-nginx sasm_docker

# If it works, test it
curl http://localhost:8080

# Clean up
docker stop test-nginx
docker rm test-nginx
```

If you get permission errors, that's okay - proceed to Assignment 3.

---

## Assignment 3: Upload to DockerHub

### Step 1: Create DockerHub Account

1. Go to https://hub.docker.com/signup
2. Create account with username format: `[firstname][lastname]sasm[year]`
   - Example for 2024-2025: `gillesmuyshondtsasm2425`
   - Example for 2025-2026: `gillesmuyshondtsasm2526`
3. Set a secure password
4. Verify your email

### Step 2: Login to DockerHub

```bash
docker login
```

Enter:
- **Username**: Your DockerHub username
- **Password**: Your DockerHub password

Expected output: `Login Succeeded`

### Step 3: Tag Your Image

Replace `[username]` with your actual DockerHub username:

```bash
docker tag sasm_docker [username]/sasm_docker:latest
```

Example:
```bash
docker tag sasm_docker gillesmuyshondtsasm2526/sasm_docker:latest
```

### Step 4: Push to DockerHub

```bash
docker push [username]/sasm_docker:latest
```

Example:
```bash
docker push gillesmuyshondtsasm2526/sasm_docker:latest
```

### Step 5: Verify Upload

Visit: `https://hub.docker.com/r/[username]/sasm_docker`

You should see your repository listed as public!

---

## Assignment 4: Fix Docker Compose File

### The Original (Broken) File

```yaml
version: '3'
service:
   nextcloud:
    image: nextcloud
    restart: always
    ports:
      - 8080:8080
    volumes:
      - "nextcloud":/var/www/html
    environments:
      - POSTGRES_HOST=db
      - POSTGRES_DB=nextcloud
      - POSTGRES_USER=nextcloud
      - POSTGRES_PASSWORD=nextcloud
      - NEXTCLOUD_ADMIN_USER=admin
      - NEXTCLOUD_ADMIN_PASS=verysecretpass
  postgres:
    image: postgres
    restart: always
    ports:
      - 5432:5432
    volumes:
    - db:/var/lib/pgsql/data
    environments:
      - POSTGRES_DB=nextcloud
      - POSTGRES_USER=nextcloud
      - POSTGRES_PASSWORD=verysecretpass
volume:
  db:
```

### Issues Identified

| Issue | Line | Problem | Solution |
|-------|------|---------|----------|
| 1 | 2 | `service:` | Should be `services:` (plural) |
| 2 | 8 | `environments:` | Should be `environment:` (no 's') |
| 3 | 20 | `environments:` | Should be `environment:` (no 's') |
| 4 | 25 | `volume:` | Should be `volumes:` (plural) |
| 5 | 7 | `8080:8080` | Should be `8080:80` (Nextcloud uses port 80) |
| 6 | 9 | `"nextcloud":/var/www/html` | Remove quotes: `nextcloud:/var/www/html` |
| 7 | 22 | `/var/lib/pgsql/data` | Should be `/var/lib/postgresql/data` |
| 8 | 14 | `NEXTCLOUD_ADMIN_PASS` | Should be `NEXTCLOUD_ADMIN_PASSWORD` |
| 9 | 11 | `POSTGRES_HOST=db` | Should be `POSTGRES_HOST=postgres` (match service name) |
| 10 | 26 | Missing `nextcloud:` volume | Need to define nextcloud volume |
| 11 | - | No `depends_on` | Should add dependency to ensure postgres starts first |
| 12 | 16,24 | Password mismatch | Both should use same password |

### The Fixed File

Create the directory and file:

```bash
mkdir -p ~/docker-compose-assignment
cd ~/docker-compose-assignment
nano docker-compose.yml
```

Paste this corrected version:

```yaml
version: '3'

services:
  nextcloud:
    image: nextcloud
    restart: always
    ports:
      - 8080:80
    volumes:
      - nextcloud:/var/www/html
    environment:
      - POSTGRES_HOST=postgres
      - POSTGRES_DB=nextcloud
      - POSTGRES_USER=nextcloud
      - POSTGRES_PASSWORD=nextcloud
      - NEXTCLOUD_ADMIN_USER=admin
      - NEXTCLOUD_ADMIN_PASSWORD=verysecretpass
    depends_on:
      - postgres

  postgres:
    image: postgres
    restart: always
    ports:
      - 5432:5432
    volumes:
      - db:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=nextcloud
      - POSTGRES_USER=nextcloud
      - POSTGRES_PASSWORD=nextcloud

volumes:
  db:
  nextcloud:
```

Save: `Ctrl+X`, then `Y`, then `Enter`

### Test the Fixed File (Optional)

**Note**: May fail on restricted servers due to permission issues.

```bash
# Start the services
docker-compose up -d

# Check if running
docker-compose ps

# Stop the services
docker-compose down
```

---

## Commit to Git

### Add all files to your repository:

```bash
# Go to your docker assignment directory
cd ~/docker-assignment

# Add files
git add Dockerfile index.html

# If you have the docker-compose file
cd ~/docker-compose-assignment
git add docker-compose.yml

# Commit with descriptive message
git commit -m "Completed Docker assignments: Dockerfile with nginx, index.html with video URLs, and fixed docker-compose.yml"

# Push to remote
git push
```

---

## Troubleshooting

### Issue: "permission denied" when building with Alpine

**Problem**: Your server runs in a restricted environment (LXC container) that doesn't support nested containerization properly.

**Solution**: Use `FROM nginx:alpine` instead of `FROM alpine` + installing nginx. This uses a pre-built image that doesn't require package installation during build.

### Issue: "permission denied" when running containers

**Problem**: Your Docker environment doesn't have full privileges to run containers.

**Solution**: Skip the testing step - you only need to **build** and **upload** the image. The assignment doesn't require running it.

### Issue: "denied: requested access to the resource is denied" when pushing

**Problem**: You're not logged in to DockerHub, or logged in with wrong username.

**Solutions**:
1. Run `docker login` and enter correct credentials
2. Make sure the username in your tag matches your logged-in username
3. Check: `docker login` shows correct username

### Issue: Container name already in use

**Problem**: A previous failed container is still registered.

**Solution**:
```bash
docker rm -f [container_name]
# or
docker rm [container_name]
```

---

## Summary Checklist

- [ ] Assignment 1: Watched 2+ Docker tutorial videos
- [ ] Assignment 2: Created Dockerfile with nginx displaying video URLs
- [ ] Assignment 2: Built Docker image successfully
- [ ] Assignment 3: Created DockerHub account
- [ ] Assignment 3: Uploaded image to DockerHub
- [ ] Assignment 4: Fixed docker-compose.yml with all 12 issues corrected
- [ ] Committed all files to Git repository

---

## Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [NGINX Docker Image](https://hub.docker.com/_/nginx)

---

## Academic Year Username Reference

| Academic Year | Username Format | Example |
|---------------|-----------------|---------|
| 2023-2024 | [name]sasm2324 | slimmeriksasm2324 |
| 2024-2025 | [name]sasm2425 | gillesmuyshondtsasm2425 |
| 2025-2026 | [name]sasm2526 | gillesmuyshondtsasm2526 |
| 2026-2027 | [name]sasm2627 | gillesmuyshondtsasm2627 |

Replace `[name]` with your first and last name without spaces or special characters.
