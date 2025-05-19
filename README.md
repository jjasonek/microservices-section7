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
