# Udemy Course Microservices with Spring Boot, Docker, Kubernetes Section7
https://www.udemy.com/course/master-microservices-with-spring-docker-kubernetes/
## spring version: 3.4.5
## Java 21


## Documentation
After adding the library, the swagger page is accessible through address 
http://localhost:8080/swagger-ui/index.html,
http://localhost:8090/swagger-ui/index.html,
http://localhost:9000/swagger-ui/index.html.


## Adding MySQL database containers

docker run -p 3306:3306 --name accountsdb -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=accountsdb -d mysql
docker run -p 3307:3306 --name loansdb -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=loansdb -d mysql
docker run -p 3308:3306 --name cardsdb -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=cardsdb -d mysql

docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                               NAMES
0caac335c78e   mysql     "docker-entrypoint.s…"   8 minutes ago    Up 8 minutes    33060/tcp, 0.0.0.0:3308->3306/tcp   cardsdb
dcba25616a18   mysql     "docker-entrypoint.s…"   10 minutes ago   Up 10 minutes   33060/tcp, 0.0.0.0:3307->3306/tcp   loansdb
360e86cae28b   mysql     "docker-entrypoint.s…"   31 minutes ago   Up 31 minutes   0.0.0.0:3306->3306/tcp, 33060/tcp   accountsdb


## Update Docker Compose files to use theMySQL DB

### regenerate images
cd ..\accounts\
mvn compile jib:dockerBuild
cd ..\cards\
mvn compile jib:dockerBuild
cd ..\loans\
mvn compile jib:dockerBuild
cd ..\configserver\
mvn compile jib:dockerBuild

PS C:\Training\Microservices\section7\configserver> docker images
REPOSITORY                                 TAG               IMAGE ID       CREATED         SIZE
...
jjasonek/loans                             s7                8db789bb4aa8   55 years ago    362MB
jjasonek/cards                             s7                2089b8cc8fd2   55 years ago    362MB
jjasonek/configserver                      s7                f70a201defbf   55 years ago    331MB
jjasonek/accounts                          s7                cc115ee4240a   55 years ago    362MB

docker image push docker.io/jjasonek/accounts:s7
docker image push docker.io/jjasonek/loans:s7
docker image push docker.io/jjasonek/cards:s7
docker image push docker.io/jjasonek/configserver:s7

### Test the docker compose
cd .\default\
docker compose up -d

docker ps
CONTAINER ID   IMAGE                      COMMAND                  CREATED       STATUS                 PORTS                               NAMES
dbee89073782   jjasonek/accounts:s7       "java -cp @/app/jib-…"   2 hours ago   Up 2 hours             0.0.0.0:8080->8080/tcp              accounts-ms
5460a9d4b0f6   mysql                      "docker-entrypoint.s…"   2 hours ago   Up 2 hours (healthy)   33060/tcp, 0.0.0.0:3308->3306/tcp   cardsdb
9cd4e3265fbc   mysql                      "docker-entrypoint.s…"   2 hours ago   Up 2 hours (healthy)   33060/tcp, 0.0.0.0:3307->3306/tcp   loansdb
a4fb1d649922   jjasonek/configserver:s7   "java -cp @/app/jib-…"   2 hours ago   Up 2 hours (healthy)   0.0.0.0:8071->8071/tcp              configserver-ms
893d5888d6d1   mysql                      "docker-entrypoint.s…"   2 hours ago   Up 2 hours (healthy)   0.0.0.0:3306->3306/tcp, 33060/tcp   accountsdb

### loans-ms and cards-ms did not start.
