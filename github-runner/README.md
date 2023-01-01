Based off amazing work at: https://github.com/knatnetwork/github-runner

## Quick pointers:

1. The packages in this repo expose the docker images.
2. The docker-compose uses those images to deploy the runners
3. Also `docker-compose.yml` will need a PAT 
4. The deployment is using `docker swarm`. Swarm mode needs following:
    1. `docker swarm init`
    2. `docker stack deploy -c docker-compose.yml <stack-name. Ex: gh-self-hosted-runner>`
    3. To stop the stack: `docker stack rm <stack-name>`
    4. To scale a service, use following after starting the stack `docker service scale <service-name, ex: runner>=<number of instances, ex:10>`
5. Swarm is important so since for ephemeral mode, swarm will create a new instance and deploy it
6. The final deployment is done as following :
```
$ DOCKER_HOST="ssh://root@github-runner-hetzner" docker stack deploy -c docker-compose.yml gh-self-hosted-runner
$ DOCKER_HOST="ssh://root@github-runner-hetzner" docker service scale gh-self-hosted-runner_runner=10
```
