## To run the docker-compose:

### Pipe based deployment
```bash
curl -fsSL https://raw.githubusercontent.com/Aimed-Labs/docker/master/nginx-with-acme-ssl/docker-compose.yml | docker-compose -f - -p 'aimed-labs_nginx-with-acme-ssl' up
```

### Compose-file as intermediate step
```bash
curl -fsSL https://raw.githubusercontent.com/Aimed-Labs/docker/master/nginx-with-acme-ssl/docker-compose.yml > docker-compose.yml && docker-compose -f ./docker-compose.yml -p 'aimed-labs_nginx-with-acme-ssl' up
```

### In github-action workflows
```yaml
  deploy-aimed-labs-nginx-proxy:
    runs-on: ubuntu-latest
    steps:
      - name: "Get the source code for proxy"
        run: |
          git clone --depth=1 https://github.com/Aimed-Labs/docker

      - name: "Deploy proxy"
        uses: aimed-labs/deploy-docker-compose-action@main
        with:
         working-directory: ./docker/nginx-with-acme-ssl
         project-name: 'aimed-labs_nginx-with-acme-ssl'
         host-address: ${{ secrets.<server-domain-name-or-ip> }}
         host-ssh-user: ${{ secrets.<server-user-that-is-accesible> }}
         filter: 'label=AIMED_LABS=true'
         ssh-private-key-base64: ${{ secrets.<private-key-in-base64> }}
```


