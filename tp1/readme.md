# remise à zéro
docker stop 3334c508b9b8
docker rm 3334c508b9b8

# build
docker build -t laurinebou/postgre .

# run
docker login
docker network create net
docker run -d --name postgre --network net laurinebou/postgre
docker run -d -p 8080:8080 --name db_view --network net adminer

# initialisation bd dans fichier docker
*création fichier sql
COPY sql/01-create.sql /docker-entrypoint-initdb.d
COPY sql/02-init.sql /docker-entrypoint-initdb.d
# Arrêt des contenaires + rebuild
docker rm -f db_view
docker rm -f postgre
docker build -t laurinebou/postgre .
docker run -d --name postgre --network net laurinebou/postgre
docker run -d -p 8080:8080 --name db_view --network net adminer

# perist data
docker rm -f postgre
docker run -d --name postgre -v /tmp/data:/var/lib/postgresql/data --network net laurinebou/postgre

# backend api
*création fichier java
*dockerfile
COPY Main.class /java
RUN java Main 

# build
docker rm -f 0f270ba5f35ddc4c943e435569a366269794b155554fb8e19f8c53a4e9c1f8df
javac Main.java
docker build -t laurinebou/java .
docker run -d --name java laurinebou/java

# multistage build
*création dockerfile spring
docker rm -f db_view
docker build -t laurinebou/spring .
docker run --name spring -p 8080:8080 laurinebou/spring     (ne pas mettre l'option -d)

# backend api
docker rm -f spring
docker build -t laurinebou/api .
docker run -d --name spring -p 8080:8080 laurinebou/spring
