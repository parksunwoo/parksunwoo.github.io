---
title: "Django 그리고 Postgres, Gunicorn, Nginx Dockerizing"
categories:
  - Docker
tags:
  - django
  - postgres
  - gunicorn
  - nginx
  - howto
last_modified_at: 2021-05-29T14:06:00-00:00
---

Django-Postgres-Gunicorn-Nginx 구성환경을 Docker 에 구현하는 튜토리얼을 참고해서 작성한 글입니다.

---
<br>

현재 회사에서 개발중인 시스템 환경자체도 도커로 구성되어 있고

연결되는 클라우드 환경도 도커파일로 구성되어 있다.

도커 이야기를 많이 듣게 되는데 도커가 왜 필요한지 어떤 점이 좋아서 많이 사용되는지를 알아야 할 것이다.

우선 도커파일을 한번 잘 구성해두면 환경설정을 다시 할 필요가 없다. 원래 환경설정을 다시 할 필요가 없지 않냐고 반문할 수 있겠지만 프로젝트는 혼자서 코딩하는게 아니라 다른 개발자와도 협업을 해야한다. 다른 개발자가 프로젝트에 참여했을때 쉽게 접근할 수 있어야 하는데 기존에는 환경 셋업을 하는데 정말 많은 시간이 들어갔다. 도커환경은 독립된 별도의 공간이어서 도커파일을 실행하는 명령어를 따라하면 쉽게 환경을 구성하고 프로젝트에 착수 할 수 있다.

도커 컴포즈를 사용하는 이유

도커파일은 하나의 환경을 기준으로 한다. 실제로 환경을 구축해보면 장고 백엔드 환경만으로는 부족하다. nginx 를 추가해야할 수도 있고 redis 를 추가해야할 수도 있다. 

구별된 여러개의 도커파일을  한곳에서 제어하는게 도커 컴포즈 파일이다.

도커 컴포즈 파일에서는 여러개의 도커환경을 서로 연결해서 구성하고 저장공간 그리고 의존관계를 설정할 수 있다.

이하 튜토리얼은 [Dockerizing Django with Postgres, Gunicorn, and Nginx](https://testdriven.io/blog/dockerizing-django-with-postgres-gunicorn-and-nginx/) 해당 튜토리얼을 참고해서 작성했습니다

---

*환경설정*

1. Django v3.0.7
2. Docker v19.03.8
3. Python v3.8.3

# 프로젝트 설정

```bash
$ mkdir django-on-docker && cd django-on-docker
$ mkdir app && cd app
$ python3.8 -m venv env
$ source env/bin/activate
(env)$ pip install django==3.0.7
(env)$ django-admin.py startproject hello_django .
(env)$ python manage.py migrate
(env)$ python manage.py runserver

```

runserver 실행후 [http://localhost:8000/](http://localhost:8000/) 에 접속하면 장고 화면이 나옴을 확인할 수 있다. 
간단히 동작하는 장고 프로젝트를 만든 것이다

app 폴더내에 requirements.txt 파일을 생성하고  장고 패키지 정보를 입력합니다.

```
Django==3.0.7
```

# 도커

[도커를 설치](https://docs.docker.com/get-docker/)하고(링크를 따라 Docker Desktop 을 설치할 것을 권장합니다), 역시 app 폴더에 Dockerfile  을 추가합니다.

```docker
# python 3.8.3 이미지를 베이스 이미지로 합니다
FROM python:3.8.3-alpine

# 작업용 디렉토리를 지정합니다
WORKDIR /usr/src/app

# 환경 변수를 설정합니다
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# 패키지들을 설치합니다
RUN pip install --upgrade pip
COPY ./requirements.txt .
RUN pip install -r requirements.txt

# 호스트상의 프로젝트 파일들을 이미지 안에 복사합니다
COPY . .
```

1. PYTHONDONTWRITEBYTECODE : 파이썬은 소스 모듈을 임포트 할 때 .pyc 파일을 쓰려고 하지 않습니다 (python -B 옵션과 동일)
2. PYTHONUNBUFFERED : stdout 과 stderr 스트림을 버퍼링하지 않도록 만듭니다 (python -u옵션과 동일)

다음으로 docker-compose.yml 파일을 프로젝트 경로에 생성합니다

```yaml
version: '3.7'

services:
  web:
    build: ./app
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - ./app/:/usr/src/app/
    ports:
      - 8000:8000
    env_file:
      - ./.env.dev
```

[settings.py](http://settings.py) 파일에서 SECRET_KEY, DEBUG, ALLOWED_HOSTS 세개 변수를 아래와 같이 수정합니다

```python
SECRET_KEY = os.environ.get("SECRET_KEY")

DEBUG = int(os.environ.get("DEBUG", default=0))

# 'DJANGO_ALLOWED_HOSTS' 는 허용할 호스트 값의 문자열 값으로 스페이스 간격을 기준으로 자릅니다
# 예를들면, 'DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]'
ALLOWED_HOSTS = os.environ.get("DJANGO_ALLOWED_HOSTS").split(" ")
```

.env.dev 파일을 만들어서 위에서 선언한 변수들에 들어갈 값을 입력합니다

```python
DEBUG=1
SECRET_KEY=foo
DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
```

차례로 이미지를 빌드하고 컨테이너를 실행합니다

```bash
$ docker-compose build
$ docker-compose up -d
```

위 명령어 실행후 [http://localhost:8000/](http://localhost:8000/) 에 접속하면 장고 화면이 나옴을 확인할 수 있다. 

# Postgres

postgres 설정을 위해선 docker-compose.yml 파일에 db 서비스를 추가해야합니다. Django 셋팅을 업데이트하기 위해선 Psycopg2 해당 패키지를 설치해야 합니다

docker-compose.yml 파일에 db 라는 서비스를 추가했습니다

```yaml
version: '3.7'

services:
  web:
    build: ./app
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - ./app/:/usr/src/app/
    ports:
      - 8000:8000
    env_file:
      - ./.env.dev
    depends_on:
      - db
  db:
    image: postgres:12.0-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    environment:
      - POSTGRES_USER=hello_django
      - POSTGRES_PASSWORD=hello_django
      - POSTGRES_DB=hello_django_dev

volumes:
  postgres_data:
```

컨테이너가 종료되어도 db 데이터를 지속하기 위해 volumes 설정을 추가했습니다
해당 설정은 postgres_data 로 명명되어 컨테이너의 /var/lib/postgresql/data/ 해당 경로에 데이터가 저장됩니다

.env.dev 파일에 아래와 같이 web 서비스에서 사용될 새로운 환경변수들을 추가합니다

```python
DEBUG=1
SECRET_KEY=foo
DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
## 추가된 부분
SQL_ENGINE=django.db.backends.postgresql
SQL_DATABASE=hello_django_dev
SQL_USER=hello_django
SQL_PASSWORD=hello_django
SQL_HOST=db
SQL_PORT=5432
```

환경설정 파일을 사용할 [settings.py](http://settings.py) 의 DATABASES 부분도 알맞게 수정합니다

환경설정 파일에서 해당하는 환경변수 값을 불러오고 해당값이 없을때는 뒤에 있는 디폴트값을 사용합니다.

```python
DATABASES = {
    "default": {
        "ENGINE": os.environ.get("SQL_ENGINE", "django.db.backends.sqlite3"),
        "NAME": os.environ.get("SQL_DATABASE", os.path.join(BASE_DIR, "db.sqlite3")),
        "USER": os.environ.get("SQL_USER", "user"),
        "PASSWORD": os.environ.get("SQL_PASSWORD", "password"),
        "HOST": os.environ.get("SQL_HOST", "localhost"),
        "PORT": os.environ.get("SQL_PORT", "5432"),
    }
}
```

Psycopg2 를 설치할 수있게 추가 패키지들을 설치해야하고 도커파일에 해당부분을 업데이트합니다

```python
# 공식 베이지 이미지를 pull 합니다
FROM python:3.8.3-alpine

# 작업공간을 설정합니다
WORKDIR /usr/src/app

# 환경 변수를 설정합니다
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# psycopg2 dependencies 설치합니다
RUN apk update \
    && apk add postgresql-dev gcc python3-dev musl-dev

# requirements 를 설치합니다
RUN pip install --upgrade pip
COPY ./requirements.txt .
RUN pip install -r requirements.txt

# 프로젝트 소스를 복사합니다
COPY . .
```

requirements.txt 에 Psycopg2 관련 패키지를 추가합니다

```
Django==3.0.7
psycopg2-binary==2.8.5
```

변경된 새로운 이미지를 빌드하고 2개 컨테이너 (services 하위에 있는 web, db)를 실행시킵니다

```bash
$ docker-compose up -d --build
```

web  서비스내에서 migrate 작업을 실행시킵니다

```bash
$ docker-compose exec web python manage.py migrate --noinput
```

아래와 같은 에러가 발생한다면 정상입니다

데이터베이스 내에  hello_django_dev 라는 데이터베이스가 없기때문입니다

```bash
django.db.utils.OperationalError: FATAL:  database "hello_django_dev" does not existv
```

아래 명령어로 컨테이너와 함께 volume를 제거합니다

```bash

$ docker-compose down -v
```

db 컨테이너에 dbname 은 hello_django_dev, username 은 hello_django 로 기본 장고 테이블을 설정합니다

```bash
$ docker-compose exec db psql --username=hello_django --dbname=hello_django_dev
```

아래명령어로 volume 이 잘생성되었고 동작하는지 확인해볼수있습니다

```bash
$ docker volume inspect django-on-docker_postgres_data
```

```json
[
    {
        "CreatedAt": "2020-06-13T18:43:56Z",
        "Driver": "local",
        "Labels": {
            "com.docker.compose.project": "django-on-docker",
            "com.docker.compose.version": "1.25.4",
            "com.docker.compose.volume": "postgres_data"
        },
        "Mountpoint": "/var/lib/docker/volumes/django-on-docker_postgres_data/_data",
        "Name": "django-on-docker_postgres_data",
        "Options": null,
        "Scope": "local"
    }
]
```

app 폴더에 아래 [entrypoint.sh](http://entrypoint.sh/) 을 추가해서 migration을 적용하기 전과 장고 개발서버를 실행시키기 전에 Postgres가 정상인지 확인할 수 있습니다

```bash
#!/bin/sh

if [ "$DATABASE" = "postgres" ]
then
    echo "Waiting for postgres..."

    while ! nc -z $SQL_HOST $SQL_PORT; do
      sleep 0.1
    done

    echo "PostgreSQL started"
fi

python manage.py flush --no-input
python manage.py migrate

exec "$@"
```

해당 스크립트의 권한을 변경해서 동작 가능하게 합니다

```bash
$ chmod +x app/entrypoint.sh
```

도커파일 가장아래에 entrypoint 명령어를 사용해 [entrypoint.sh](http://entrypoint.sh/) 을 실행시키는 것을 추가합니다

```python
# 공식 베이지 이미지를 pull 합니다
FROM python:3.8.3-alpine

# 작업공간을 설정합니다
WORKDIR /usr/src/app

# 환경 변수를 설정합니다
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# psycopg2 dependencies 설치합니다
RUN apk update \
    && apk add postgresql-dev gcc python3-dev musl-dev

# requirements 를 설치합니다
RUN pip install --upgrade pip
COPY ./requirements.txt .
RUN pip install -r requirements.txt

# 프로젝트 소스를 복사합니다
COPY . .

# run entrypoint.sh
ENTRYPOINT ["/usr/src/app/entrypoint.sh"]
```

.env.dev  파일에  DATABASE 변수를 추가합니다

```
DEBUG=1
SECRET_KEY=foo
DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
SQL_ENGINE=django.db.backends.postgresql
SQL_DATABASE=hello_django_dev
SQL_USER=hello_django
SQL_PASSWORD=hello_django
SQL_HOST=db
SQL_PORT=5432
DATABASE=postgres ## 추가한 부분
```

다시 테스트를 해봅니다

1. 이미지 재빌드
2. 컨테이너 실행
3. [http://localhost:800](http://localhost:800) 접속

# 노트

첫번째, Postgres 를 더했음에도 불구하고, 장고를 위한 독립적인 도커이미지를 생성할  수 있습니다

DATABASE 환경변수를 postgres 로 설정하지 않는한, 테스트를 위해 새로운 이미지를 빌드하고 컨테이너를 실행해봅니다

```bash
$ docker build -f ./app/Dockerfile -t hello_django:latest ./app
$ docker run -d \
    -p 8006:8000 \
    -e "SECRET_KEY=please_change_me" -e "DEBUG=1" -e "DJANGO_ALLOWED_HOSTS=*" \
    hello_django python /usr/src/app/manage.py runserver 0.0.0.0:8000
```

[http://localhost:8006](http://localhost:8006) 에 접속하면 장고 페이지를 볼수있습니다

두번째,  [entrypoint.sh](http://entrypoint.sh/) 에서 데이터베이스를 flush, migrate 하는 명령어를 주석처리해서 모든 컨테이너가 시작또는 재시작할때마다 실행되지 않게 합니다

```bash
#!/bin/sh

if [ "$DATABASE" = "postgres" ]
then
    echo "Waiting for postgres..."

    while ! nc -z $SQL_HOST $SQL_PORT; do
      sleep 0.1
    done

    echo "PostgreSQL started"
fi

# python manage.py flush --no-input
# python manage.py migrate

exec "$@"
```

대신에 수동으로 컨테이너가 실행된 다음에 직접 실행시킬수있습니다 

```bash
$ docker-compose exec web python manage.py flush --no-input
$ docker-compose exec web python manage.py migrate
```

# Gunicorn

운영환경으로 가기위해선 Gunicorn 을 더해야합니다.

```
Django==3.0.7
gunicorn==20.0.4
psycopg2-binary==2.8.5
```

개발용으로 장고 내장서버를 사용하기를 원하기 때문에 운영용으로 docker-compose.prod.yml 컴포즈 파일을 만듭니다

```yaml
version: '3.7'

services:
  web:
    build: ./app
    command: gunicorn hello_django.wsgi:application --bind 0.0.0.0:8000
    ports:
      - 8000:8000
    env_file:
      - ./.env.prod
    depends_on:
      - db
  db:
    image: postgres:12.0-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    env_file:
      - ./.env.prod.db

volumes:
  postgres_data:
```

command 에 주목해보면 이제는 장고 개발서버가 아닌 gunicorn으로 동작하게 됩니다. 또한 web servic에서 volume 을 제거했습니다, 운영 환경에서 더이상 필요하지 않기 떄문입니다

마지막으로  web 과 db에서 구분된 환경변수 파일을 사용해 컨테이너가 run 될때에 각각 다른 환경변수 파일이 사용됩니다. (./.env.prod 그리고 ./.env.prod.db)

*.env.prod*:

```python
DEBUG=0
SECRET_KEY=change_me
DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
SQL_ENGINE=django.db.backends.postgresql
SQL_DATABASE=hello_django_prod
SQL_USER=hello_django
SQL_PASSWORD=hello_django
SQL_HOST=db
SQL_PORT=5432
DATABASE=postgres
```

*.env.prod.db*:

```yaml
POSTGRES_USER=hello_django
POSTGRES_PASSWORD=hello_django
POSTGRES_DB=hello_django_prod
```

두개 환경변수 파일은 프로젝트 경로에 위치하고 버전관리에서 제외하고 싶다면 .gitignore 파일에 추가하면 됩니다

-v 플래그를 사용해 개발 컨테이너를 종료시킵니다

```yaml
$ docker-compose down -v
```

운영용 이미지를 빌드하고  나서 실행시키는 명령어 입니다

```yaml
$ docker-compose -f docker-compose.prod.yml up -d --build
```

장고 기본 테이블로 hello_django_prod 가 제대로 생성되었는지 확인해봅니다. 테스트는 [http://localhost:8000/admin](http://localhost:8000/admin) 어드민 페이지 에서 확인합니다. 정적 파일들이 아직 로딩되지않았음을 확인할 수 있습니다. 디버그 모드가 꺼져있었기때문에 예상할 수 있었던 부분입니다. 간단히 고쳐보겠습니다.

컨테이너에서 에러가 발생했을때 에러 로그를 통해 체크할 수 있습니다.

```yaml
$ docker-compose -f docker-compose.prod.yml logs -f
```

# 운영용 도커파일

매번 컨테이너를 실행할떄마다 flush 와 migrate 명령어가 동작하던것을 기억하시나요?

개발환경에서는 괜찮지만 운영환경을 위해선 새로운 entrypoint 파일을 생성해야합니다.

[entrypoint.prod.sh](http://entrypoint.prod.sh/):

```bash
#!/bin/sh

if [ "$DATABASE" = "postgres" ]
then
    echo "Waiting for postgres..."

    while ! nc -z $SQL_HOST $SQL_PORT; do
      sleep 0.1
    done

    echo "PostgreSQL started"
fi

exec "$@"
```

파일 권한을 수정합니다

```bash
$ chmod +x app/entrypoint.prod.sh
```

위 파일을 사용해 운영환경에서 사용된 새로운 도커파일 [Dockerfile.prod](http://dockerfile.prod) 를 생성합니다 

```bash
###########
# BUILDER #
###########

# 공식 베이스 이미지를 pull
FROM python:3.8.3-alpine as builder

# 작업 공간설정
WORKDIR /usr/src/app

# 환경변수 설정
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# psycopg2 디펜던시 설치
RUN apk update \
    && apk add postgresql-dev gcc python3-dev musl-dev

# lint
RUN pip install --upgrade pip
RUN pip install flake8
COPY . .
RUN flake8 --ignore=E501,F401 .

# 디펜던시 설치
COPY ./requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /usr/src/app/wheels -r requirements.txt

#########
# FINAL #
#########

# 공식 베이스 이미지를 pull
FROM python:3.8.3-alpine

# app user를 위한 폴더 생성
RUN mkdir -p /home/app

# app user 생성
RUN addgroup -S app && adduser -S app -G app

# 적절한 디렉토리 생성
ENV HOME=/home/app
ENV APP_HOME=/home/app/web
RUN mkdir $APP_HOME
WORKDIR $APP_HOME

# 디펜던시 설치
RUN apk update && apk add libpq
COPY --from=builder /usr/src/app/wheels /wheels
COPY --from=builder /usr/src/app/requirements.txt .
RUN pip install --no-cache /wheels/*

# entrypoint-prod.sh 복사
COPY ./entrypoint.prod.sh $APP_HOME

# 프로젝트 파일 복사
COPY . $APP_HOME

# app user 모든 파일 권한변경
RUN chown -R app:app $APP_HOME

# app user 변경
USER app

# entrypoint.prod.sh 실행
ENTRYPOINT ["/home/app/web/entrypoint.prod.sh"]
```

여기에서 우리는 최종 이미지 사이즈를 줄이기위해 multi-stage 빌드 도커를 사용합니다. 
builder 는 Python wheels 가 빌드되는 동안 사용되는 임시적인 이미지 입니다. wheels 는 최종 운영 이미지에 복사되고 나서 builder 이미지는 무시됩니다. 

root 가 아닌 유저를 생성한 것을 기억하는가? 기본적으로 도커는 컨테이너 내부적으로는 root 계정과 같이 컨테이너 프로세스를 동작시킨다. 공격자가 도커 호스트의 root 권한을 갖을 수 있게 하는 것은 컨테이너 관리를 함에 있어서 좋지 않은 사례이다. 컨테이너 안에서 root 권한을 갖는다면 호스트 상에서 root 일 것이다.

docker-compose.prod.yml 파일의 web 서비스 부분에서 사용하는 도커파일을 [Dockerfile.prod](http://dockerfile.prod) 로 수정합니다

```yaml
web:
  build:
    context: ./app
    dockerfile: Dockerfile.prod
  command: gunicorn hello_django.wsgi:application --bind 0.0.0.0:8000
  ports:
    - 8000:8000
  env_file:
    - ./.env.prod
  depends_on:
    - db
```

실행해봅니다.

```bash
$ docker-compose -f docker-compose.prod.yml down -v
$ docker-compose -f docker-compose.prod.yml up -d --build
$ docker-compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput
```

# Nginx

예를 들면 static file의 전송과 같은 클라이언트 요청을 제어하는 gunocorn 의 reverse proxy 서버 역할로 nginx를 추가해봅니다. 

docker-compose.prod.yml 에 서비스를 더합니다

```yaml
nginx:
  build: ./nginx
  ports:
    - 1337:80
  depends_on:
    - web
```

프로젝트 경로에 nginx 폴더를 생성한후 해당 폴더 안에 Dockerfile, nginx.conf 파일을 생성합니다

도커파일은

```docker
FROM nginx:1.19.0-alpine

RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d
```

nginx.conf 는

```
upstream hello_django {
    server web:8000;
}

server {

    listen 80;

    location / {
        proxy_pass http://hello_django;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }

}
```

docker-compose.prod.yml 파일의 web 서비스에서 ports 를 expose 로 교체합니다.

```yaml
web:
  build:
    context: ./app
    dockerfile: Dockerfile.prod
  command: gunicorn hello_django.wsgi:application --bind 0.0.0.0:8000
  expose:
    - 8000
  env_file:
    - ./.env.prod
  depends_on:
    - db
```

이제 8000번 포트는 내부적으로만 다른 도커 서비스에 노출됩니다. 포트는 더이상 호스트 머신에서 공개되지 않습니다.

다시 테스트 해봅니다.

```
$ docker-compose -f docker-compose.prod.yml down -v
$ docker-compose -f docker-compose.prod.yml up -d --build
$ docker-compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput
```

web 서비스는 8000 포트에 열려있고 nginx.conf 에서 8000포트는 80번 포트에 맵핑, docker-compose.prod.yml 파일에서 nginx 는 80번 포트가 1337 포트에 맵핑되어 있습니다.

따라서 외부에서 접속할 수 있는 포트는 1337 포트입니다

[http://localhost:1337](http://localhost:1337/) 접속되는걸 확인할 수 있습니다

컨테이너를 종료시킵니다.

```yaml
$ docker-compose -f docker-compose.prod.yml down -v
```

Gunicorn은 application 서버이기 때문에 static file을 전송하지 않습니다. 

그럼 static, media 파일을 어떻게 다룰 수 있을까요?

# Static Files

[settings.py](http://settings.py) 를 수정합니다

```
STATIC_URL = "/staticfiles/"
STATIC_ROOT = os.path.join(BASE_DIR, "staticfiles")
```

## 개발환경

이제 해당 경로의 어느 요청이든   [`http://localhost:8000/staticfiles/*](http://localhost:8000/staticfiles/*)` 

staticfiles 폴더에서 파일이 서빙됩니다

테스트를 위해 최초의 이미지를 재빌드하고 새로운 컨테이너를 실행시켜봅니다

[http://localhost:8000/admin](http://localhost:8000/admin) 접속해보면 static 파일이 정확히 서빙되는 것을 확인할 수 있습니다

## 운영환경

운영환경을 위해  docker-compose.prod.yml 에  있는 web 과 nginx 서비스에 볼륨을 추가합니다.

각 컨테이너는 staticfiles 라는 이름의 디렉토리를 서로 공유할 수 있습니다

 

```yaml
version: '3.7'

services:
  web:
    build:
      context: ./app
      dockerfile: Dockerfile.prod
    command: gunicorn hello_django.wsgi:application --bind 0.0.0.0:8000
    volumes:
      - static_volume:/home/app/web/staticfiles
    expose:
      - 8000
    env_file:
      - ./.env.prod
    depends_on:
      - db
  db:
    image: postgres:12.0-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    env_file:
      - ./.env.prod.db
  nginx:
    build: ./nginx
    volumes:
      - static_volume:/home/app/web/staticfiles
    ports:
      - 1337:80
    depends_on:
      - web

volumes:
  postgres_data:
  static_volume:
```

당신은 또한 [Dockerfile.prod](http://dockerfile.prod) 파일에서 /home/app/web/staticfiles 폴더를 생성해야합니다.

```yaml
...

# 적절한 경로를 생성합니다
ENV HOME=/home/app
ENV APP_HOME=/home/app/web
RUN mkdir $APP_HOME
RUN mkdir $APP_HOME/staticfiles
WORKDIR $APP_HOME

...
```

이러한 작업이 필요한 이유는 무엇일까요? 
도커 컴포즈는 일반적으로 root 사용자로써 볼륨을 마운트하는데 
현재 우리가 사용하고 있는 root 가 아닌 사용자인 경우,  권한문제가 발생해 collectstatic 명령어가 동작하지 않을 수 있습니다

해당 이슈를 해결하려면 아래와 같은 방법을 사용할 수있는데

1. 도커파일 안에 폴더를 생성한다
2. 마운트 된 폴더의 권한을 변경한다

여기서는 1번 방법을 사용해보겠습니다

다음으로 Nginx 설정에서 static file  요청이 라우팅 되는 staticfiles 폴더 설정을 수정합니다

```yaml
upstream hello_django {
    server web:8000;
}

server {

    listen 80;

    location / {
        proxy_pass http://hello_django;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }

    location /staticfiles/ {
        alias /home/app/web/staticfiles/;
    }

}
```

개발계 컨테이너를 잠시 내리고

테스트를 해봅니다

```yaml
$ docker-compose -f docker-compose.prod.yml up -d --build
$ docker-compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput
$ docker-compose -f docker-compose.prod.yml exec web python manage.py collectstatic --no-input --clear
```

다시  [http://localhost:1337/staticfiles/*](http://localhost:1337/staticfiles/*) 접속하면 staticfiles 폴더에서 파일이 서빙됩니다

[http://localhost:1337/admin](http://localhost:1337/admin) 어드민 페이지에 접속했을때에도 static 파일들이 정확히 로딩 되는 것을 확인할 수 있습니다

또한 `docker-compose -f docker-compose.prod.yml logs -f`

로그를 통해서도 요청에서 static 파일들이 Nginx 를 통해 성공적으로 서빙됨을 확인할 수 있습니다

```yaml
nginx_1  | 172.31.0.1 - - [13/Jun/2020:20:35:47 +0000] "GET /admin/ HTTP/1.1" 302 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36" "-"
nginx_1  | 172.31.0.1 - - [13/Jun/2020:20:35:47 +0000] "GET /admin/login/?next=/admin/ HTTP/1.1" 200 1928 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36" "-"
nginx_1  | 172.31.0.1 - - [13/Jun/2020:20:35:47 +0000] "GET /staticfiles/admin/css/base.css HTTP/1.1" 304 0 "http://localhost:1337/admin/login/?next=/admin/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36" "-"
nginx_1  | 172.31.0.1 - - [13/Jun/2020:20:35:47 +0000] "GET /staticfiles/admin/css/login.css HTTP/1.1" 304 0 "http://localhost:1337/admin/login/?next=/admin/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36" "-"
nginx_1  | 172.31.0.1 - - [13/Jun/2020:20:35:47 +0000] "GET /staticfiles/admin/css/responsive.css HTTP/1.1" 304 0 "http://localhost:1337/admin/login/?next=/admin/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36" "-"
nginx_1  | 172.31.0.1 - - [13/Jun/2020:20:35:47 +0000] "GET /staticfiles/admin/css/fonts.css HTTP/1.1" 304 0 "http://localhost:1337/admin/login/?next=/admin/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36" "-"
nginx_1  | 172.31.0.1 - - [13/Jun/2020:20:35:47 +0000] "GET /staticfiles/admin/fonts/Roboto-Regular-webfont.woff HTTP/1.1" 304 0 "http://localhost:1337/staticfiles/admin/css/fonts.css" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36" "-"
nginx_1  | 172.31.0.1 - - [13/Jun/2020:20:35:47 +0000] "GET /staticfiles/admin/fonts/Roboto-Light-webfont.woff HTTP/1.1" 304 0 "http://localhost:1337/staticfiles/admin/css/fonts.css" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36" "-"
```

컨테이너를 다시 종료합니다

```yaml
$ docker-compose -f docker-compose.prod.yml down -v
```

# Media Files

media 파일을 다루는걸 테스트하기 위해선 Django 의 새로운 앱을 생성해야합니다

```yaml
$ docker-compose up -d --build
$ docker-compose exec web python manage.py startapp upload
```

[settings.py](http://settings.py) 파일에서 `INSTALLED_APPS` 새롭게 생성한 upload 를 추가합니다

```yaml
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",

    "upload",
]
```

app/upload/views.py 을 아래와 같이 작성합니다

```python
from django.shortcuts import render
from django.core.files.storage import FileSystemStorage

def image_upload(request):
    if request.method == "POST" and request.FILES["image_file"]:
        image_file = request.FILES["image_file"]
        fs = FileSystemStorage()
        filename = fs.save(image_file.name, image_file)
        image_url = fs.url(filename)
        print(image_url)
        return render(request, "upload.html", {
            "image_url": image_url
        })
    return render(request, "upload.html")
```

"app/upload" 폴더에 "templates" 폴더를 생성하고 새로운 template 파일 upload.html 을 추가합니다

```html
{% block content %}

  <form action="{% url "upload" %}" method="post" enctype="multipart/form-data">
    {% csrf_token %}
    <input type="file" name="image_file">
    <input type="submit" value="submit" />
  </form>

  {% if image_url %}
    <p>File uploaded at: <a href="{{ image_url }}">{{ image_url }}</a></p>
  {% endif %}

{% endblock %}
```

app/hello_django/urls.py

[urls.py](http://urls.py) 파일을 아래와 같이 생성합니다

```python
from django.contrib import admin
from django.urls import path
from django.conf import settings
from django.conf.urls.static import static

from upload.views import image_upload

urlpatterns = [
    path("", image_upload, name="upload"),
    path("admin/", admin.site.urls),
]

if bool(settings.DEBUG):
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

app/hello_django/settings.py:

[settings.py](http://settings.py) 에 MEDIA 와 관련된 설정을 추가합니다

```python
MEDIA_URL = "/mediafiles/"
MEDIA_ROOT = os.path.join(BASE_DIR, "mediafiles")
```

## 개발환경

테스트를 위해 

```yaml
$ docker-compose up -d --build
```

이제 [http://localhost:8000/](http://localhost:8000/) 에서 이미지를 업로드할 수 있고

업로드한 이미지를 [http://localhost:8000/mediafiles/IMAGE_FILE_NAME](http://localhost:8000/mediafiles/IMAGE_FILE_NAME) 에서 확인할 수 있습니다

## 운영환경

운영환경을 위해서 web 과 nginx 서비스에 또다른 볼륨을 추가합니다

```yaml
version: '3.7'

services:
  web:
    build:
      context: ./app
      dockerfile: Dockerfile.prod
    command: gunicorn hello_django.wsgi:application --bind 0.0.0.0:8000
    volumes:
      - static_volume:/home/app/web/staticfiles
      - media_volume:/home/app/web/mediafiles
    expose:
      - 8000
    env_file:
      - ./.env.prod
    depends_on:
      - db
  db:
    image: postgres:12.0-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    env_file:
      - ./.env.prod.db
  nginx:
    build: ./nginx
    volumes:
      - static_volume:/home/app/web/staticfiles
      - media_volume:/home/app/web/mediafiles
    ports:
      - 1337:80
    depends_on:
      - web

volumes:
  postgres_data:
  static_volume:
  media_volume:
```

Dockerfile.prod 파일에 "/home/app/web/mediafiles" 폴더를 생성합니다  

```yaml
...

# 적절한 폴더를 생성합니다
ENV HOME=/home/app
ENV APP_HOME=/home/app/web
RUN mkdir $APP_HOME
RUN mkdir $APP_HOME/staticfiles
RUN mkdir $APP_HOME/mediafiles
WORKDIR $APP_HOME

...
```

Nginx 설정을 수정합니다

```yaml
upstream hello_django {
    server web:8000;
}

server {

    listen 80;

    location / {
        proxy_pass http://hello_django;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }

    location /staticfiles/ {
        alias /home/app/web/staticfiles/;
    }
		
		# 추가된 부분
    location /mediafiles/ {
        alias /home/app/web/mediafiles/;
    }

}
```

다시 빌드합니다

```
$ docker-compose down -v

$ docker-compose -f docker-compose.prod.yml up -d --build
$ docker-compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput
$ docker-compose -f docker-compose.prod.yml exec web python manage.py collectstatic --no-input --clear
```

최종적으로 확인해보면

1. [http://localhost:1337/](http://localhost:1337/) 에서 이미지를 업로드합니다
2. [http://localhost:1337/mediafiles/IMAGE_FILE_NAME](http://localhost:1337/mediafiles/IMAGE_FILE_NAME) 에서 이미지를 확인합니다

혹시 413 Request Entity Too Large 에러를 만나게된다면 Nginx 설정에서 클라이언트 request body 에서 허용되는 최대파일의 크기를 변경하면 됩니다

예를 들면,

```yaml
location / {
    proxy_pass http://hello_django;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;
    proxy_redirect off;
    client_max_body_size 100M; # 추가된 부분
}
```