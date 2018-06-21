```bash
# Deploy app to demo server
docker stack deploy registry -c docker-compose.yml
```

The app should now be available at
http://nginx.${DEMO_SERVER_HOSTNAME}