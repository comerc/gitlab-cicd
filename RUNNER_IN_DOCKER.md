Вот пример конфигурации `docker-compose.yml` для GitLab Runner:

```yaml
version: '3.5'
services:
  dind:
    image: docker:20-dind
    restart: always
    privileged: true
    environment:
      DOCKER_TLS_CERTDIR: ''
    command:
      - --storage-driver=overlay2
  runner:
    restart: always
    image: registry.gitlab.com/gitlab-org/gitlab-runner:alpine
    depends_on:
      - dind
    environment:
      - DOCKER_HOST=tcp://dind:2375
    volumes:
      - ./config:/etc/gitlab-runner:z
  register-runner:
    restart: 'no'
    image: registry.gitlab.com/gitlab-org/gitlab-runner:alpine
    depends_on:
      - dind
    environment:
      - CI_SERVER_URL=${CI_SERVER_URL}
      - REGISTRATION_TOKEN=${REGISTRATION_TOKEN}
    command:
      - register
      - --non-interactive
      - --locked=false
      - --name=${RUNNER_NAME}
      - --executor=docker
      - --docker-image=docker:20-dind
      - --docker-volumes=/var/run/docker.sock:/var/run/docker.sock
    volumes:
      - ./config:/etc/gitlab-runner:z
```

Вам также потребуется файл `.env` со следующим содержимым:

```bash
RUNNER_NAME=RUNNER-NAME
REGISTRATION_TOKEN=TOKEN
CI_SERVER_URL=https://gitlab.com/
```

Этот пример позволяет запускать задания GitLab CI/CD в отдельных, изолированных контейнерах Docker. Пожалуйста, замените `RUNNER_NAME`, `REGISTRATION_TOKEN`, и `CI_SERVER_URL` на соответствующие значения для вашего проекта.
