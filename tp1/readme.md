# remise a zero
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
* creation fichier sql
COPY sql/01-create.sql /docker-entrypoint-initdb.d
COPY sql/02-init.sql /docker-entrypoint-initdb.d
# Arret des contenaires + rebuild
docker rm -f db_view
docker rm -f postgre
docker build -t laurinebou/postgre .
docker run -d --name postgre --network net laurinebou/postgre
docker run -d -p 8080:8080 --name db_view --network net adminer

# perist data
docker rm -f postgre
docker run -d --name postgre -v /tmp/data:/var/lib/postgresql/data --network net laurinebou/postgre

# backend api
* creation fichier java
* dockerfile
COPY Main.class /java
RUN java Main 

# build
docker rm -f 0f270ba5f35ddc4c943e435569a366269794b155554fb8e19f8c53a4e9c1f8df
javac Main.java
docker build -t laurinebou/java .
docker run -d --name java laurinebou/java

# multistage build
* creation dockerfile spring
docker rm -f db_view
docker build -t laurinebou/spring .
docker run --name spring -p 8080:8080 laurinebou/spring     (ne pas mettre l'option -d)

# backend api
docker rm -f spring
* dans dossier sql
docker build -t laurinebou/postgre .
docker run -d --name postgre --network net laurinebou/postgre
* dans dossier backend
docker build -t laurinebou/api .
docker run --name api -p 8080:8080 --network net laurinebou/api

# http server
docker rm -f api  
docker rm -f postgre
* dockerfile
FROM httpd:2.4
COPY index.html /usr/local/apache2/htdocs/
* dans dossier http
docker build -t laurinebou/http .
docker run -d --name http -p 80:80 --network net laurinebou/http
docker run --rm httpd:2.4 cat /usr/local/apache2/conf/httpd.conf > my-httpd.conf
* dans dockerfile
FROM httpd:2.4
COPY ./my-httpd.conf /usr/local/apache2/conf/httpd.conf

# reverse proxy
* run postgre
docker build -t laurinebou/postgre .
docker run -d --name postgre -v /tmp/data:/var/lib/postgresql/data --network net laurinebou/postgre
* run backend
docker build -t laurinebou/backend .
docker run -d --name backend -p 8080:8080 --network net laurinebou/backend
* run http
docker build -t laurinebou/http .
docker run -d --name http -p 80:80 --network net laurinebou/http

# link application
*  fichier docker-composer et 
docker-compose up --build

docker-compose down