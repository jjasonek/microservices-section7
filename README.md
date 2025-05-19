# Udemy Course Microservices with Spring Boot, Docker, Kubernetes Section6
https://www.udemy.com/course/master-microservices-with-spring-docker-kubernetes/
## spring version: 3.4.5
## Java 21


## Documentation
After adding the library, the swagger page is accessible through address 
http://localhost:8080/swagger-ui/index.html,
http://localhost:8090/swagger-ui/index.html,
http://localhost:9000/swagger-ui/index.html.


## Configuration management v1-springboot

### Environment
This: envirinment.getProperty("JAVA_HOME")
gives output C:\Java\jdk-17.0.7 when I run the application from IntelliJ, 
which is strange since I changed the property to C:\Java\jdk-21.0.1

### @ConfigurationProperties
set of properties with prefix "accounts"

result of GET
{
    "message": "Welcome to EazyBank accounts related local APIs",
    "contactDetails": {
        "name": "John Doe - Developer",
        "email": "john@eazybank.com"
},
    "onCallSupport": [
        "(555) 555-1234",
        "(555) 523-1345"
    ]
}



## Configuration management v2-spring-cloud-config

## classpath approach
After start of the Configserver we can se properties from urls beginning with localhost:8071
For example localhost:8071/accounts/prod gives:
{
    "name": "accounts",
    "profiles": [
    "prod"
    ],
    "label": null,
    "version": null,
    "state": null,
    "propertySources": [
        {
            "name": "classpath:/config/accounts-prod.yml",
            "source": {
                "build.version": "1.0",
                "accounts.message": "Welcome to EazyBank accounts related prod APIs",
                "accounts.contactDetails.name": "Reine Aishwarya - Product Owner",
                "accounts.contactDetails.email": "aishwarya@eazybank.com",
                "accounts.onCallSupport[0]": "(453) 392-4829",
                "accounts.onCallSupport[1]": "(236) 203-0384"
            }
        },
        {
            "name": "classpath:/config/accounts.yml",
            "source": {
                "build.version": "3.0",
                "accounts.message": "Welcome to EazyBank accounts related local APIs",
                "accounts.contactDetails.name": "John Doe - Developer",
                "accounts.contactDetails.email": "john@eazybank.com",
                "accounts.onCallSupport[0]": "(555) 555-1234",
                "accounts.onCallSupport[1]": "(555) 523-1345"
            }
        }
    ]
}

### Starting the accounts application with the qa profile:
2025-05-13T15:08:16.595+02:00  INFO 14392 --- [accounts] [  restartedMain] c.e.accounts.AccountsApplication         : The following 1 profile is active: "qa"
2025-05-13T15:08:16.643+02:00  INFO 14392 --- [accounts] [  restartedMain] o.s.c.c.c.ConfigServerConfigDataLoader   : Fetching config from server at : http://localhost:8071
2025-05-13T15:08:16.643+02:00  INFO 14392 --- [accounts] [  restartedMain] o.s.c.c.c.ConfigServerConfigDataLoader   : Located environment: name=accounts, profiles=[default], label=null, version=null, state=null
2025-05-13T15:08:16.643+02:00  INFO 14392 --- [accounts] [  restartedMain] o.s.c.c.c.ConfigServerConfigDataLoader   : Fetching config from server at : http://localhost:8071
2025-05-13T15:08:16.643+02:00  INFO 14392 --- [accounts] [  restartedMain] o.s.c.c.c.ConfigServerConfigDataLoader   : Located environment: name=accounts, profiles=[qa], label=null, version=null, state=null


### Retrieving the properties from the Configserver I can see the following:
2025-05-13T21:10:03.776+02:00  INFO 40280 --- [configserver] [nio-8071-exec-1] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: Config resource 'file [C:\Users\jijas\AppData\Local\Temp\config-repo-11059085055692324310\accounts.yml]' via location 'file:/C:/Users/jijas/AppData/Local/Temp/config-repo-11059085055692324310/'
2025-05-13T21:10:03.778+02:00  WARN 40280 --- [configserver] [nio-8071-exec-1] o.s.c.c.s.e.CipherEnvironmentEncryptor   : Cannot decrypt key: accounts.contactDetails.email (class java.lang.UnsupportedOperationException: No decryption for FailsafeTextEncryptor. Did you configure the keystore correctly?)
2025-05-13T21:10:04.309+02:00  WARN 40280 --- [configserver] [nio-8071-exec-2] o.s.c.c.s.e.EnvironmentController        : Error getting the Environment with name=.well-known profiles=appspecific label=com.chrome.devtools.json includeOrigin=false
...
Caused by: org.eclipse.jgit.api.errors.RefNotFoundException: Ref com.chrome.devtools.json cannot be resolved

I think the reason is settings in the GitHub side.
Like email in accounts-prod.yml 


## Refreshing the config properties using the refresh endpoint in actuator

### Now we cen see, our properties on the Config Server are updating.
Like here: http://localhost:8071/accounts/prod

### To refresh properties for the microservices without restarting them 
we need to call the actuator refresh endpoint: 
POST http://localhost:8080/actuator/refresh for the accounts.

And the response is:
[
    "config.client.version",
    "accounts.message"
]

2025-05-14T16:03:04.606+02:00  INFO 30468 --- [configserver] [nio-8071-exec-6] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: Config resource 'file [C:\Users\jijas\AppData\Local\Temp\config-repo-2333009254686404261\accounts.yml]' via location 'file:/C:/Users/jijas/AppData/Local/Temp/config-repo-2333009254686404261/'
2025-05-14T16:03:05.135+02:00  INFO 30468 --- [configserver] [nio-8071-exec-7] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: Config resource 'file [C:\Users\jijas\AppData\Local\Temp\config-repo-2333009254686404261\accounts-prod.yml]' via location 'file:/C:/Users/jijas/AppData/Local/Temp/config-repo-2333009254686404261/'
2025-05-14T16:03:05.135+02:00  INFO 30468 --- [configserver] [nio-8071-exec-7] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: Config resource 'file [C:\Users\jijas\AppData\Local\Temp\config-repo-2333009254686404261\accounts.yml]' via location 'file:/C:/Users/jijas/AppData/Local/Temp/config-repo-2333009254686404261/'

2025-05-14T16:03:04.058+02:00  INFO 44180 --- [accounts] [nio-8080-exec-3] o.s.c.c.c.ConfigServerConfigDataLoader   : Fetching config from server at : http://localhost:8071
2025-05-14T16:03:04.608+02:00  INFO 44180 --- [accounts] [nio-8080-exec-3] o.s.c.c.c.ConfigServerConfigDataLoader   : Located environment: name=accounts, profiles=[default], label=null, version=082cd1ce7f42f9a5b0d918fb959d46729883a52d, state=
2025-05-14T16:03:04.621+02:00  INFO 44180 --- [accounts] [nio-8080-exec-3] o.s.c.c.c.ConfigServerConfigDataLoader   : Fetching config from server at : http://localhost:8071
2025-05-14T16:03:05.137+02:00  INFO 44180 --- [accounts] [nio-8080-exec-3] o.s.c.c.c.ConfigServerConfigDataLoader   : Located environment: name=accounts, profiles=[prod], label=null, version=082cd1ce7f42f9a5b0d918fb959d46729883a52d, state=
2025-05-14T16:03:06.173+02:00  INFO 44180 --- [accounts] [nio-8080-exec-3] o.s.cloud.commons.util.InetUtils         : Cannot determine local hostname
2025-05-14T16:03:06.176+02:00  INFO 44180 --- [accounts] [nio-8080-exec-3] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
2025-05-14T16:03:06.397+02:00  INFO 44180 --- [accounts] [nio-8080-exec-3] o.s.cloud.endpoint.RefreshEndpoint       : Refreshed keys : [config.client.version, accounts.message]


## Refreshing the config properties using Spring Cloud Bus

### Install/run RabbitMQ
~$ docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4-management
...
2025-05-15 12:10:34.297314+00:00 [info] <0.499.0> epmd monitor knows us, inter-node communication (distribution) port: 25672
...
2025-05-15 12:10:34.595369+00:00 [info] <0.216.0> Created user 'guest'
2025-05-15 12:10:34.597655+00:00 [info] <0.216.0> Successfully set user tags for user 'guest' to [administrator]
2025-05-15 12:10:34.600440+00:00 [info] <0.216.0> Successfully set permissions for user 'guest' in virtual host '/' to '.*', '.*', '.*'


### After configuration, and after updating properties in the GitHub, call POST http://localhost:8080/actuator/bus-refresh 

2025-05-15T15:21:31.289+02:00  INFO 34468 --- [configserver] [NisFliEpciFEA-1] o.s.cloud.bus.event.RefreshListener      : Received remote refresh request.
2025-05-15T15:21:31.975+02:00  INFO 34468 --- [configserver] [nio-8071-exec-3] .c.s.e.MultipleJGitEnvironmentRepository : Fetched for remote main and found 1 updates

2025-05-15T15:21:31.255+02:00  INFO 5552 --- [loans] [EyF9lw0xW9Qnw-1] o.s.cloud.bus.event.RefreshListener      : Received remote refresh request.
2025-05-15T15:21:31.277+02:00  INFO 5552 --- [loans] [EyF9lw0xW9Qnw-1] o.s.c.c.c.ConfigServerConfigDataLoader   : Fetching config from server at : http://localhost:8071
2025-05-15T15:21:33.007+02:00  INFO 5552 --- [loans] [EyF9lw0xW9Qnw-1] o.s.c.c.c.ConfigServerConfigDataLoader   : Located environment: name=loans, profiles=[default], label=null, version=94cb0196ceea47070c4210da0ed274dd8f50d885, state=
2025-05-15T15:21:33.019+02:00  INFO 5552 --- [loans] [EyF9lw0xW9Qnw-1] o.s.c.c.c.ConfigServerConfigDataLoader   : Fetching config from server at : http://localhost:8071
2025-05-15T15:21:34.355+02:00  INFO 5552 --- [loans] [EyF9lw0xW9Qnw-1] o.s.c.c.c.ConfigServerConfigDataLoader   : Located environment: name=loans, profiles=[prod], label=null, version=94cb0196ceea47070c4210da0ed274dd8f50d885, state=

2025-05-15T15:21:31.257+02:00  INFO 13568 --- [cards] [aGZmXB9d0Z67Q-1] o.s.cloud.bus.event.RefreshListener      : Received remote refresh request.
2025-05-15T15:21:31.280+02:00  INFO 13568 --- [cards] [aGZmXB9d0Z67Q-1] o.s.c.c.c.ConfigServerConfigDataLoader   : Fetching config from server at : http://localhost:8071
2025-05-15T15:21:32.549+02:00  INFO 13568 --- [cards] [aGZmXB9d0Z67Q-1] o.s.c.c.c.ConfigServerConfigDataLoader   : Located environment: name=cards, profiles=[default], label=null, version=94cb0196ceea47070c4210da0ed274dd8f50d885, state=
2025-05-15T15:21:32.562+02:00  INFO 13568 --- [cards] [aGZmXB9d0Z67Q-1] o.s.c.c.c.ConfigServerConfigDataLoader   : Fetching config from server at : http://localhost:8071
2025-05-15T15:21:33.461+02:00  INFO 13568 --- [cards] [aGZmXB9d0Z67Q-1] o.s.c.c.c.ConfigServerConfigDataLoader   : Located environment: name=cards, profiles=[prod], label=null, version=94cb0196ceea47070c4210da0ed274dd8f50d885, state=

2025-05-15T15:21:31.181+02:00  INFO 24392 --- [accounts] [io-8080-exec-10] o.s.cloud.bus.event.RefreshListener      : Received remote refresh request.
2025-05-15T15:21:31.206+02:00  INFO 24392 --- [accounts] [io-8080-exec-10] o.s.c.c.c.ConfigServerConfigDataLoader   : Fetching config from server at : http://localhost:8071
2025-05-15T15:21:32.023+02:00  INFO 24392 --- [accounts] [io-8080-exec-10] o.s.c.c.c.ConfigServerConfigDataLoader   : Located environment: name=accounts, profiles=[default], label=null, version=94cb0196ceea47070c4210da0ed274dd8f50d885, state=
2025-05-15T15:21:32.039+02:00  INFO 24392 --- [accounts] [io-8080-exec-10] o.s.c.c.c.ConfigServerConfigDataLoader   : Fetching config from server at : http://localhost:8071



## Refreshing the config properties using Spring Cloud Bus & Spring Cloud Monitor

$ docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4-management

$ docker run --entrypoint /bin/sh --rm -it hookdeck/hookdeck-cli
/ # hookdeck login --cli-key 3hmd3ov6j5838dkp5sn37dd2ufe4t2pnbvr5bsgp0lt9lblueo
> The Hookdeck CLI is configured with your console Sandbox
/ #  hookdeck listen host.docker.internal:8071 Source --cli-path /monitor

Dashboard
👉 Inspect and replay events: https://dashboard.hookdeck.com

Sources
🔌 Source URL: https://hkdk.events/7s76jlue5z64je

Connections
Source -> Source_to_cli-Source forwarding to /monitor

> Ready! (^C to quit)
2025-05-17 18:30:41 [200] POST http://host.docker.internal:8071/monitor | https://dashboard.hookdeck.com/cli/events/evt_bYThg0TEW5a2RQABoO
2025-05-17 18:32:44 [200] POST http://host.docker.internal:8071/monitor | https://dashboard.hookdeck.com/cli/events/evt_CjhXXhNsK9j2igcpkQ
2025-05-17 18:33:32 [200] POST http://host.docker.internal:8071/monitor | https://dashboard.hookdeck.com/cli/events/evt_nZicKl7QWRMkjpwhvR


## Actuator API calls after adding health configuration in application.yml

GET http://localhost:8071/actuator/health
{
    "status": "DOWN",
    "groups": [
        "liveness",
        "readiness"
    ]
}

http://localhost:8071/actuator/health/liveness
{
    "status": "UP"
}

http://localhost:8071/actuator/health/readiness
{
    "status": "UP"
}


## Optimization of docker-compose.yml

- We move repeating stuff to common-config.yml.
- We created 3 services: network-deploy-service is only for rabbit, 
   microservice-base-config is for the configserver, 
   microservice-configserver-config is for the rest of the microservices.
- In recent versions of Docker Compose it's no longer possible to mention 
  extends and depends_on directives in within a service.

  
## Build Docker Images

### build
docker login
PS C:\Training\Microservices\section6\v2-spring-cloud-config\accounts> mvn compile jib:dockerBuild
...
[INFO] Built image to Docker daemon as jjasonek/accounts:s6

PS C:\Training\Microservices\section6\v2-spring-cloud-config\cards> mvn compile jib:dockerBuild
...
[INFO] Built image to Docker daemon as jjasonek/cards:s6

PS C:\Training\Microservices\section6\v2-spring-cloud-config\loans> mvn compile jib:dockerBuild
...
[INFO] Built image to Docker daemon as jjasonek/loans:s6

PS C:\Training\Microservices\section6\v2-spring-cloud-config\configserver> mvn compile jib:dockerBuild
...
[INFO] Built image to Docker daemon as jjasonek/configserver:s6

### check and clean
docker images
REPOSITORY                                 TAG            IMAGE ID       CREATED         SIZE
jjasonek/accounts                          s4             a25eb3d0b593   2 weeks ago     503MB
...
rabbitmq                                   4-management   2b9b9a8fbf0e   4 weeks ago     275MB
...
jjasonek/loans                             s4             ff8c9def766e   45 years ago    310MB
jjasonek/loans                             s6             f8cf2d3d4391   55 years ago    381MB
jjasonek/cards                             s4             040ba03ca6b8   55 years ago    351MB
jjasonek/accounts                          s6             fe157fd58338   55 years ago    381MB
jjasonek/cards                             s6             cc1922012a1c   55 years ago    381MB
jjasonek/configserver                      s6             521c6f9deccc   55 years ago    348MB

docker rmi a25eb3d0b593 ff8c9def766e 040ba03ca6b8 2b9b9a8fbf0e

docker images
REPOSITORY                                 TAG       IMAGE ID       CREATED         SIZE
...
jjasonek/accounts                          s6        fe157fd58338   55 years ago    381MB
jjasonek/cards                             s6        cc1922012a1c   55 years ago    381MB
jjasonek/loans                             s6        f8cf2d3d4391   55 years ago    381MB
jjasonek/configserver                      s6        521c6f9deccc   55 years ago    348MB

### push to docker hub
docker image push docker.io/jjasonek/accounts:s6
docker image push docker.io/jjasonek/loans:s6
docker image push docker.io/jjasonek/cards:s6
docker image push docker.io/jjasonek/configserver:s6


## Testing

docker compose up -d

### hookdeck
docker run --entrypoint /bin/sh --rm -it hookdeck/hookdeck-cli
hookdeck login --cli-key 3hmd3ov6j5838dkp5sn37dd2ufe4t2pnbvr5bsgp0lt9lblueo
hookdeck listen host.docker.internal:8071 Source --cli-path /monitor

### verify
docker ps
CONTAINER ID   IMAGE                      COMMAND                  CREATED       STATUS                 PORTS                                                                                                         NAMES
c1fadff7b116   hookdeck/hookdeck-cli      "/bin/sh"                2 hours ago   Up 2 hours                                                                                                                           charming_nightingale
5313430b1982   jjasonek/cards:s6          "java -cp @/app/jib-…"   2 hours ago   Up 2 hours             0.0.0.0:9000->9000/tcp                                                                                        cards-ms
9667346c2268   jjasonek/loans:s6          "java -cp @/app/jib-…"   2 hours ago   Up 2 hours             0.0.0.0:8090->8090/tcp                                                                                        loans-ms
71e594011da1   jjasonek/accounts:s6       "java -cp @/app/jib-…"   2 hours ago   Up 2 hours             0.0.0.0:8080->8080/tcp                                                                                        accounts-ms
0b0f176af625   jjasonek/configserver:s6   "java -cp @/app/jib-…"   2 hours ago   Up 2 hours (healthy)   0.0.0.0:8071->8071/tcp                                                                                        configserver-ms
bd898d05e90c   rabbitmq:3.13-management   "docker-entrypoint.s…"   2 hours ago   Up 2 hours (healthy)   4369/tcp, 5671/tcp, 0.0.0.0:5672->5672/tcp, 15671/tcp, 15691-15692/tcp, 25672/tcp, 0.0.0.0:15672->15672/tcp   default-rabbit-1

### Test updating config failed, because our configserver (and other microservices) tries to connect to rabbitmq on localhost. 
But it is not true because we have all services in their containers.
### hookdeck log:
2025-05-19 18:26:26 [500] POST http://host.docker.internal:8071/monitor | https://dashboard.hookdeck.com/cli/events/evt_sOworaLS7nhqObTt0z

docker logs -f 0b0f176af625

### After overriding property SPRING_RABBITMQ_HOST: "rabbit" in common-config.yml:
### confogserver log:
2025-05-19T18:55:27.828Z  INFO 1 --- [configserver] [           main] o.s.c.s.binder.DefaultBinderFactory      : Creating binder: rabbit
2025-05-19T18:55:27.828Z  INFO 1 --- [configserver] [           main] o.s.c.s.binder.DefaultBinderFactory      : Constructing binder child context for rabbit
2025-05-19T18:55:27.976Z  INFO 1 --- [configserver] [           main] o.s.c.s.binder.DefaultBinderFactory      : Caching the binder: rabbit
2025-05-19T18:55:28.000Z  INFO 1 --- [configserver] [           main] c.s.b.r.p.RabbitExchangeQueueProvisioner : declaring queue for inbound: springCloudBus.anonymous.ieYz8-1JQFygSoDQjkzNJg, bound to: springCloudBus
2025-05-19T18:55:28.004Z  INFO 1 --- [configserver] [           main] o.s.a.r.c.CachingConnectionFactory       : Attempting to connect to: [rabbit:5672]
2025-05-19T18:55:28.041Z  INFO 1 --- [configserver] [           main] o.s.a.r.c.CachingConnectionFactory       : Created new connection: rabbitConnectionFactory#ce19c86:0/SimpleConnection@5625ba2 [delegate=amqp://guest@172.20.0.2:5672/, localPort=56152]
...
2025-05-19T18:55:32.353Z  INFO 1 --- [configserver] [nio-8071-exec-2] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: Config resource 'file [/tmp/config-repo-3869145523301761123/cards.yml]' via location 'file:/tmp/config-repo-3869145523301761123/'
2025-05-19T18:55:33.053Z  INFO 1 --- [configserver] [nio-8071-exec-4] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: Config resource 'file [/tmp/config-repo-3869145523301761123/loans.yml]' via location 'file:/tmp/config-repo-3869145523301761123/'
2025-05-19T18:55:33.532Z  INFO 1 --- [configserver] [nio-8071-exec-3] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: Config resource 'file [/tmp/config-repo-3869145523301761123/accounts.yml]' via location 'file:/tmp/config-repo-3869145523301761123/'
2025-05-19T18:57:14.899Z  INFO 1 --- [configserver] [nio-8071-exec-5] o.s.c.c.monitor.PropertyPathEndpoint     : Refresh for: accounts
2025-05-19T18:57:14.933Z  INFO 1 --- [configserver] [nio-8071-exec-5] o.s.c.s.m.DirectWithAttributesChannel    : Channel 'configserver.springCloudBusOutput' has 1 subscriber(s).
2025-05-19T18:57:14.967Z  INFO 1 --- [configserver] [nio-8071-exec-5] o.s.a.r.c.CachingConnectionFactory       : Attempting to connect to: [rabbit:5672]
2025-05-19T18:57:14.972Z  INFO 1 --- [configserver] [nio-8071-exec-5] o.s.a.r.c.CachingConnectionFactory       : Created new connection: rabbitConnectionFactory.publisher#5a94a35a:0/SimpleConnection@152603d0 [delegate=amqp://guest@172.20.0.2:5672/, localPort=36124]
2025-05-19T18:57:14.979Z  INFO 1 --- [configserver] [nio-8071-exec-5] o.s.amqp.rabbit.core.RabbitAdmin         : Auto-declaring a non-durable, auto-delete, or exclusive Queue (springCloudBus.anonymous.ieYz8-1JQFygSoDQjkzNJg) durable:false, auto-delete:true, exclusive:true. It will be redeclared if the broker stops and is restarted while the connection factory is alive, but all messages will be lost.
2025-05-19T18:57:14.996Z  INFO 1 --- [configserver] [nio-8071-exec-5] o.s.cloud.bus.event.RefreshListener      : Received remote refresh request.
2025-05-19T18:57:14.997Z  INFO 1 --- [configserver] [nio-8071-exec-5] o.s.cloud.bus.event.RefreshListener      : Refresh not performed, the event was targeting accounts:**
2025-05-19T18:57:15.819Z  INFO 1 --- [configserver] [nio-8071-exec-6] .c.s.e.MultipleJGitEnvironmentRepository : Fetched for remote main and found 1 updates
2025-05-19T18:57:15.841Z  INFO 1 --- [configserver] [nio-8071-exec-6] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: Config resource 'file [/tmp/config-repo-3869145523301761123/accounts.yml]' via location 'file:/tmp/config-repo-3869145523301761123/'

GET http://localhost:8071/actuator/health
{
    "status": "UP",
    "groups": [
        "liveness",
        "readiness"
    ]
}

### hookdeck log:
2025-05-19 19:08:59 [200] POST http://host.docker.internal:8071/monitor | https://dashboard.hookdeck.com/cli/events/evt_TR8MMJDiaVO9Zu1j9n

docker compose down


## Other environments

### testing prod:
cd ..\prod\
docker compose up -d

And the rest is the same.
