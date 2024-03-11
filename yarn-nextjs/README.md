### apk repo config

```dockerfile
RUN apk add --repository=http://somthing1.com \
    --repository=http://somthing2.com \
    --no-cache libc6-compat logrotate
```

### docker build args in jenkins

```bash
$ --build-arg PHASE={{phase}} --build-arg RELEASE_VERSION=$(git rev-parse HEAD) --build-arg= PROJECT_NAME=john
```
