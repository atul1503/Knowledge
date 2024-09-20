in docker compose you can share volume by declaring a volume name in `volumes:` section and using it in the volumes section of each service. like:


declare the volume:

`
volumes:
  redis_data:
`
and then use that volume in any container:

`
redis:
    image: "redis:latest"
    volumes:
      - redis_data:/data
`

here `redis_data` volume is mounted at /data directory of the redis service container.

**dont push files in shared volume in dockerfile . To push files in shared volume, use the facilities provided after docker image build like in entrypoint of docker compose or command in docker compose**. Like this:

`
services:
  videocaption:
    build:
      context: .
      dockerfile: Dockerfile_django 
    env_file:
      - videocaption.env
    entrypoint: /app/server_init.sh
    volumes:
      - project_code:/volume
    depends_on:
      - postgres
`

where server_init.sh is :

`
#!/bin/bash

cd /volume
cp -r /app/api /app/videocaption /app/manage.py /app/build /app/media .
python3 manage.py makemigrations
python3 manage.py makemigrations api 
python3 manage.py migrate
python3 manage.py runserver 0.0.0.0:8000
`
here the shared volume is mounted at /volume
this script is running after the image is already built using dockerfile.
