# Skill Test : 1 for Docker and Containers
This application provides details on testing various services after running the docker-compose file. These services include User, Product, Order, and Gateway Services. Each service has its own endpoints for testing purposes.

## Services, ports and end poins 
-  User
-  Product
-  Order
-  Gateway Services

### Ports and end points
#### **User Service**
- **Base URL:** `http://localhost:3000`
- **Endpoints:**
  - **List Users:**  
    ```
    curl http://localhost:3000/users
    ```
    Or open in your browser: [http://localhost:3000/users](http://localhost:3000/users)

#### **Product Service**
- **Base URL:** `http://localhost:3001`
- **Endpoints:**
  - **List Products:**  
    ```
    curl http://localhost:3001/products
    ```
    Or open in your browser: [http://localhost:3001/products](http://localhost:3001/products)

#### **Order Service**
- **Base URL:** `http://localhost:3002`
- **Endpoints:**
  - **List Orders:**  
    ```
    curl http://localhost:3002/orders
    ```
    Or open in your browser: [http://localhost:3002/orders](http://localhost:3002/orders)


#### **Gateway Service**
- **Base URL:** `http://localhost:3003/api`
- **Endpoints:**
  - **Users:**  
    ```
    curl http://localhost:3003/api/users
    ```
  - **Products:**  
    ```
    curl http://localhost:3003/api/products
    ```
  - **Orders:**  
    ```
    curl http://localhost

### Docker & Container
#### File Structure 
Microsservices/ <br>
├── user-service/ <br>
│   └── _Dockerfile_ <br>
├── product-service/ <br>
│   └── _Dockerfile_ <br>
├── order-service/ <br>
│   └── _Dockerfile_ <br>
├── gateway-service/ <br>
│   └── _Dockerfile_ <br>
├── _docker-compose.yml_ <br>
└── README.md <br>
└── document.md <br>

##### Sample Dockerfile for user service
```
FROM node:22-alpine
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3003
CMD ["node","app.js"]
```
And it is the similar Dockerfile for all other services<br>

##### docker-compose file
Docker compose file is to build all images at a time<br>
```
version: "3.9"

services:
  user-service:
    build: ./user-service
    ports:
      - "3000:3000"
    depends_on:
      - gateway-service
    networks:
      - microservices-network

  product-service:
    build: ./product-service
    ports:
      - "3001:3001"
    depends_on:
      - gateway-service
    networks:
      - microservices-network

  order-service:
    build: ./order-service
    ports:
      - "3002:3002"
    depends_on:
      - gateway-service
    networks:
      - microservices-network

  gateway-service:
    build: ./gateway-service
    ports:
      - "3003:3003"
    networks:
      - microservices-network

networks:
  microservices-network:
    driver: bridge
```
## Build and run the containers
### Docker Images after build<br>
<img width="1201" height="198" alt="image" src="https://github.com/user-attachments/assets/a76a2a1f-c072-4ab2-889b-d5abddbc108e" />

### Running Docker Containers <br>
<img width="1891" height="194" alt="image" src="https://github.com/user-attachments/assets/36821628-37be-4895-b313-d3a4eb1120d8" />

## Test
### Testing using CURL command<br>
<img width="1725" height="162" alt="image" src="https://github.com/user-attachments/assets/d2663a19-2582-4774-a839-78a26b0badd5" />

### UI Test
#### Health check for all services
<img width="588" height="131" alt="image" src="https://github.com/user-attachments/assets/dd20ab0f-a16e-43e5-adf7-4d308b30902e" /><br>
<img width="545" height="120" alt="image" src="https://github.com/user-attachments/assets/a98ed1c8-acf1-4470-aae6-36dd65d1dc79" /><br>
<img width="562" height="134" alt="image" src="https://github.com/user-attachments/assets/6c9a94d0-5511-4ae2-830f-c65ced36fe3b" /><br>
<img width="546" height="126" alt="image" src="https://github.com/user-attachments/assets/fdf1fd3f-e974-4c32-b366-c5930d7fb13b" /><br>

#### End points testing
<img width="532" height="232" alt="image" src="https://github.com/user-attachments/assets/8e698b4b-85a2-47e7-b88b-51f0805a4d6f" /><br>
<img width="563" height="270" alt="image" src="https://github.com/user-attachments/assets/f2580f52-e2d0-4c44-b79e-46a572797d6c" /><br>
<img width="561" height="143" alt="image" src="https://github.com/user-attachments/assets/136481b4-0d5c-404a-9c1a-dfbe911c7bb5" /><br>
<img width="563" height="226" alt="image" src="https://github.com/user-attachments/assets/ff02a8a0-6e22-4bec-ad47-c25e24e450c1" /><br>

## Commands used
`docker images` -- to see the list of images <br>
`docker network ls` -- to see the network  <br>
`docker build -t <image_name>` -- to build an image <br>
`docker run -d -p <port>:<port> <image_name> .` -- to run the built image on a specific port <br>
`docker-compose build` -- to build all the images at once using docker compose <br>
`docker-compose up` -- to run all the containers with built images <br>
`docker ps` - to list all the running containers <br>
`docker exec it <container_id> /bin/bash` -- to go into a container <br>

## Basic Troubleshooting
If a docker image is build and if it is not running, you can check the logs with complete information
`docker logs <container_id`

If the docker image is build and container is running, go to the browser and launch the URL and right click and inspect
<img width="563" height="226" alt="image" src="https://github.com/user-attachments/assets/ba74262c-0695-45af-9100-06a3ba5c2228" /><br>

And go to network tab, to see exact issue
<img width="2488" height="424" alt="image" src="https://github.com/user-attachments/assets/0f8e1b94-2406-4054-8a02-8fa526925309" /><br>


