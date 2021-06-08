# Repaso basico de docker
## REPASO GENERAL

## Docker 2021

### Repaso completo. Toda la info es del Pelado Nerd de:
https://www.youtube.com/watch?v=CV_Uf3Dq-EU


### Como ejecutar una imagen, la tenga o no en mi pc:
`docker run nginx (una vez que levanta queda en consola, con Ctrl+c se corta)`

#### Como listar las imágenes:
docker images

#### Como listar las imágenes históricas:
docker images ps 
# si son muchas:
docker images ps -a | head

# Como descargar una imagen de HUB pero sin arrancarla:
docker pull nginx

# Si quiero ver que pasó con un una imagen que intenté correr y no quedó levantada (docker logs imagen):
➜  ~ docker ps -a

CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS                      PORTS     NAMES
3aa3569f32d2   postgres   "docker-entrypoint.s…"   2 minutes ago   Exited (1) 2 minutes ago              bold_turing
46510d8fd230   postgres   "docker-entrypoint.s…"   2 hours ago     Exited (0) 11 minutes ago             amazing_golick
df64a305ad46   postgres   "docker-entrypoint.s…"   2 hours ago     Exited (1) 2 hours ago                suspicious_shannon
➜  ~
➜  ~ docker logs 3aa3569f32d2
Error: Database is uninitialized and superuser password is not specified.
       You must specify POSTGRES_PASSWORD to a non-empty value for the
       superuser. For example, "-e POSTGRES_PASSWORD=password" on "docker run".

       You may also use "POSTGRES_HOST_AUTH_METHOD=trust" to allow all
       connections without a password. This is *not* recommended.

       See PostgreSQL documentation about "trust":
       https://www.postgresql.org/docs/current/auth-trust.html
➜  ~

# Como arrancar una imagen con algunas variables desde la consola:
docker run -e POSTGRES_PASSWORD=password -d postgres

# Como ejecutar una sesión interactivo (i) y emular una terminal (t):
docker exec -it (nombre contenedor) COMANDO A EJECUTAR
# Ejemplo:
docker exec -it f2017f169011 /bin/bash

# Como detener el contenedor:
docker stop Nombre_Contenedor
# Ejemplo:
➜  ~ docker ps
CONTAINER ID   IMAGE      COMMAND                  CREATED          STATUS          PORTS      NAMES
f2017f169011   postgres   "docker-entrypoint.s…"   22 minutes ago   Up 22 minutes   5432/tcp   beautiful_leakey
➜  ~ docker stop f2017f169011
f2017f169011
➜  ~ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
➜  ~

# Como hacer un build de una imagen con un Docker file y levantarlo en background en el puerto 3000:
# Ejemplo:
➜  app cat Dockerfile
FROM node:12.22.1-alpine3.11

WORKDIR /app
COPY . .
RUN yarn install --production

CMD ["node", "/app/src/index.js"]
➜  app

# Como se ejecuta: (tiene un tag para facilitar el reconocerlo)
docker build -t contenedor-prueba .

# Como se ve la imagen:
➜  app docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
➜  app docker images
REPOSITORY          TAG       IMAGE ID       CREATED              SIZE
contenedor-prueba   latest    6c096bbb93c5   About a minute ago   179MB
nginx               latest    d1a364dc548d   13 days ago          133MB
postgres            latest    293e4ed402ba   3 weeks ago          315MB

# Como levantarla en el puerto 3000 y en background:
docker run -dp 3000:3000 contenedor-prueba

# Como darle un volumen del disco que pueda tomarlo nuevamente al recrear el contenedor:
docker run -d -v /Users/rodrigogaray/Code/docker/etc:/etc/todos -p 3000:3000 contenedor-prueba

## Como crear un Entorno Multi-Container:

# Como crear una network, una red de docker de nombre todo-app:
docker network create todo-app

# Como listar las redes de docker:
docker network ls
# **(NOTA: Ojo que hay tres redes por default:
6c7eb79c1d07   bridge     bridge    local
0328bde75de2   host       host      local
7f959f392b57   none       null      local  )**

# Y a continuación la nueva:
c2939f2f3ea6   todo-app   bridge    local

# Para borrar luego la red:
docker network rm todo-app

# Nota: si quiero ver que mas puedo hacer con las redes:
docker network --help

# Contenedor de la base de datos: (las tabulaciones son al pedo)
docker run -d \
>       --network todo-app --network-alias mysql \
>       -v todo-mysql-data:/var/lib/mysql \
>       -e MYSQL_ROOT_PASSWORD=rodpincha \
>       -e MYSQL_DATABASE=todos \
>       mysql:5.7

# Contendor de la app (la app se banca las variables):
docker run -dp 3000:3000 \
> --network todo-app \
> -e MYSQL_HOST=mysql \
> -e MYSQL_USER=root \
> -e MYSQL_PASSWORD=rodpincha \
> -e MYSQL_DB=todos \
> contenedor-prueba:v1

# Veo el log del último contenedor: (se ven los contenedores)
➜  multi-container docker logs 1a9be5663c73
Waiting for mysql:3306.
Connected!
Connected to mysql db at host mysql
Listening on port 3000
➜  multi-container

## TODO LO ANTERIOR PERO HECHO EN UN DOCKER-COMPOSE SERÍA:
## ------------------------------------------------------
# El archivo Yaml:

cat docker-compose.yaml

version: "3.7"

services:

#docker run -dp 3000:3000 --network todo-app -e MYSQL_HOST=mysql -e MYSQL_USER=root -e MYSQL_PASSWORD=rodpincha -e MYSQL_DB=todos contenedor-prueba:v1

  app:
    image: contenedor-prueba:v1
    ports:
      - 3000:3000
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: rodpincha
      MYSQL_DB: todos

# docker run -d     --network todo-app --network-alias mysql     -v todo-mysql-data:/var/lib/mysql     -e MYSQL_ROOT_PASSWORD=rodpincha     -e MYSQL_DATABASE=todos     mysql:5.7

  mysql:
    image: mysql:5.7
    volumes:
      - ./todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: rodpincha
      MYSQL_DATABASE: todos

# Nota1: se ignona la parte de network porque al usar docker-compose este crea una red nueva para usar los contenedores
# Nota2: se ignora el network-alias ya que lee el nombre que se le da a cada parte y les pone ese alias, en este caso app y mysql

#Y lugo para ejecutar sería:
docker-compose up -d

## Nota: Como el mysql tarda un poco mas en levantar, la app da error, esto se ve con docker logs imagen_contendor_app
## Se vuelve a ejecutar el docker-compose -d y levanta todo lo que no esté arriba
