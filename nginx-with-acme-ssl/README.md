To run the docker-compose:

```bash
curl -fsSL https://raw.githubusercontent.com/Aimed-Labs/docker/master/nginx-with-acme-ssl/docker-compose.yml | docker-compose -f - -p 'aimed-labs_nginx-with-acme-ssl' up
```
OR
```bash
curl -fsSL https://raw.githubusercontent.com/Aimed-Labs/docker/master/nginx-with-acme-ssl/docker-compose.yml > docker-compose.yml && docker-compose -f ./docker-compose.yml -p 'aimed-labs_nginx-with-acme-ssl' up
