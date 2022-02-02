## TP1
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
* * Compliation avec JDK puis run avec JRE

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

# publish
docker tag tp1_httpd laurinebou/httpd:1.0
docker tag tp1_backend laurinebou/backend:1.0
docker tag tp1_database laurinebou/database:1.0

docker push laurinebou/httpd:1.0
docker push laurinebou/backend:1.0
docker push laurinebou/database:1.0


## TP2
# build and test your application
newgrp docker
* dans backend (pomp.xml)
mvn clean verify

* mvn clean verify : vérif qualité du code
* test unitaire : tester qu'une partie du code
* comportement test : tester indépendamment sur qu'une partie du code sans intégration
* testcontainers :permet de tester des modules défini à l'avance


## TD3

ssh -i ./id_rsa centos@laurine.boulio.takima.cloud
* fichier inventory
[dev]
centos@laurine.boulio.takima.cloud
* 
ansible all -m ping --private-key=id_rsa -i inventory.yml -u centos
ansible all -m yum -a "name=httpd state=present" --private-key=id_rsa -i inventory.yml -u centos
ansible all -m yum -a "name=httpd state=present" --private-key=id_rsa -i inventory.yml -u centos --become
ansible all -m shell -a 'echo "<html><h1>Hello CPE</h1></html>" >> /var/www/html/index.html' --private-key=id_rsa -i inventory.yml -u centos --become
ansible all -m service -a "name=httpd state=started" --private-key=id_rsa -i inventory.yml -u centos --become



## TP3
ansible all -i inventories/setup.yml -m ping

ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"

ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become

ansible-playbook -i inventories/setup.yml playbook.yml

ansible-playbook -i inventories/setup.yml playbook.yml --syntax-check

ansible-playbook -i inventories/setup.yml playbook.yml

# Creation des roles
ansible-galaxy init roles/docker
ansible-galaxy init roles/network
ansible-galaxy init roles/database
ansible-galaxy init roles/app
ansible-galaxy init roles/proxy

docker tag tp1_httpd laurinebou/httpd:1.0
docker tag tp1_backend laurinebou/backend:1.0
docker tag tp1_database laurinebou/database:1.0

docker push laurinebou/httpd:1.0
docker push laurinebou/backend:1.0
docker push laurinebou/database:1.0

ansible-playbook -i inventories/setup.yml playbook.yml


