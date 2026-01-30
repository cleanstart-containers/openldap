---
name: Container Image Issue
about: Report an issue with a container image
title: '[IMAGE NAME] '
labels: bug, image-issue
assignees: cleanstart-community-admin
---

## Image Information

**Image Name and Tag:** 
<!-- e.g., cleanstart/nginx:1.21.0-alpine -->

**Image Digest:** 
<!-- Run: docker inspect --format='{{.RepoDigests}}' IMAGE_NAME -->
<!-- e.g., sha256:abc123... -->

## Bug Description

**Describe the bug:**
<!-- A clear and concise description of what the bug is -->

**Expected behavior:**
<!-- What should happen? -->

**Actual behavior:**
<!-- What actually happens? -->

## Reproduction Steps

**Steps to reproduce the issue:**
1. Pull the image: `docker pull cleanstart/nginx:latest`
2. Run container: `docker run -d -p 8080:80 cleanstart/nginx:latest`
3. Check logs: `docker logs <container_id>`
4. See error...

**Docker Run Command / Compose File:**
```bash
# Paste your complete docker run command or docker-compose.yml
docker run -d \
  -p 8080:80 \
  -e ENV_VAR=value \
  -v /host/path:/container/path \
  cleanstart/nginx:latest
