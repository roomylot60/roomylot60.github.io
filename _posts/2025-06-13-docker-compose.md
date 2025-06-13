# Docker wrapping

## docker-compose.yml

```docker
service:
  service_name:
    build:
      context: .
      dockerfile: docker/Dockerfile.service1
    container_name: service1_container_name
    ports:
      - 8000:8000
    volumes:
      - ./dir_service1:/app
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              device_ids: ['0']
              capabilities: [gpu]
```

- `services`: 해당 docker container로 이루어진 앱에서 사용되는 기능
- `service_name`: service 명 정의(e.g. vllm-api)
- `build`:
  - `dockerfile`: container 생성에 필요한 이미지 정의서의 위치
- `container_name`: 현재 앱에서 dockerfile 혹은 docker hub에서 다운받은 이미지로 생성할 container의 이름을 정의
- `ports`: `local port number : container port number` 로 컨테이너와 연결될 port number를 정의
- `volumes`: container에 mount 시킬 디렉토리를 정의
- `device`: GPU 사용관련 정의
