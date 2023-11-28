# Build and Deploy Spring Boot Microservices using Docker

**IMAGE**

Our image will be based on the JDK image.

```python
FROM openjdk:17-oracle
```

**VOLUME**

Take example of a database running in a container. It is important to note that if we delete the container, all the data will be lost. To prevent this, we specify a data folder, in this case, "/tmp".

```python
VOLUME /tmp
```

**Generate JAR**

```python
mvn compile
mvn test 
mvn package
==
mvn clean package -DskipTests
```

Jenkins will do this work (this commands) for us after  🙂

**Run JAR**

```python
COPY target/discovery-service-0.0.1-SNAPSHOT.jar app.jar
#COPY target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Create image** 

```python
docker images
#if file_name = Dockerfile we put . else file_name
docker build . -t discovery-service:version1 
```

**Run Container** 

```python
#we can also put id "687d1ebc0bf5" instead of name "discovery-service"
docker run discovery-service:version1
```

**Stop container**

```python
#we can just put 2 or 3 chars 787d because id is unique
docker stop 687d1ebc0bf5
```

```python
#dispaly container logs
docker logs 687d1ebc0bf5
```

**Display Containers** 

```python
docker ps 
```

**Expose our container in externe**

make it accessible 

```python
docker run -p 8761:8761 discovery-service:version1
# -p port_in_externe : port_in_container
```

```python
#to run container in background not appear in console
docker run -d -p 8761:8761 discovery-service:version1
```

**Docker Compose**

Now we don’t need to use all past methods

```yaml
services:
  ebank-discovery-service:
    build: ./discovery-service                  #go to this folder to find Dockerfile
    container_name: ebank-discovery-service
    ports:
      - '8761:8761'
    expose:
      - '8761'
    healthcheck:
      #curl to send an http request to check app health
      #-f if we have a fail return fail status code
      test: ['CMD', 'curl', '-f', 'http://localhost:8761/actuator/health']
      #each 10s we send request default = 30s
      interval: 10s
      #repeat 5 times
      retries: 5
```

```python
docker compose up
#start container in background
docker compose up -d 
#if we not use --build docker will not generate an image if it already exist
docker compose up -d --build
```

**Destroy container**

this is why we use VOLUME to save data out of container

```python
docker compose down
```

```yaml
ebank-config-service:
    build: ./config-service
    container_name: ebank-config-service
    ports:
      - '9999:9999'
    expose:
      - '9999'
    environment:
      #we specify the url of ebank-discovery-service
      #we use this variable because if we use localhost docker search in it's own localhost
      - DISCOVERY_SERVICE_URL=http://ebank-discovery-service:8761/eureka
    depends_on:
      ebank-discovery-service:
        condition: service_healthy
```

lifecycle : Created → Waiting → Healthy → Started
