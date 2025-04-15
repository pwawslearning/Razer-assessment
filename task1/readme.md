## Deploy a simple web application by using Docker container with ECR private repository

![Image](https://github.com/user-attachments/assets/2bb5712e-fc6a-499e-b8e8-fe2b75ffb43a)


## 📋 Table of Contents
1. [Prerequisites](#prerequisites)
2. [Architecture](#architecture)
3. [Implementation](#implementation)
4. [Verification](#verification)

---

## Prerequisites

Before getting started, ensure you have the following:

1. **AWS account**: For deployment ECR on AWS cloud with least of privillage
2. **Install awscli and secret_key & access_key**: securely connect to AWS resources.
3. **Docker**: Docker hub account to pull public images.
---

## Architecture
- **AWS Secret manager** Create secret to connect with Docker hub by using access token.
- **IAM policy**: To pull image from upstream registry, the user who need to grant ECR registry permission.
- **ECR with Pull-Through Cache**: Configured to cache images from Docker Hub as an upstream registry.
- **Docker container**: To deploy the web server as a container.
- **Nginx Load Balancer**: To provide high availability and scalability for incoming traffic to backend servers and mediate between clients requests and servers by distributing incoming traffic equally among servers.
---

## Implementation
1. **Create aws secret with docker access token**:
  - username = docker username
  - access Token= docker registry>> account setting>>personal access tokens>>Generate new token
  ![Image](https://github.com/user-attachments/assets/3f3017df-2426-4ee6-96a3-9b640a884ccd)
  ![Image](https://github.com/user-attachments/assets/1ec4c810-739c-41be-a133-1130ad80ff7e)
  ![Image](https://github.com/user-attachments/assets/57ba323c-9e0a-4f9c-a0dd-1115575cf4b5)
2. **Define specific upstream registry and namespace**:
  ![Image](https://github.com/user-attachments/assets/198b322d-4c6e-4d77-b3c4-09bcee5a385d)
  ![Image](https://github.com/user-attachments/assets/bccc9eae-744f-4e7a-8258-a98b54794aae)
  ![Image](https://github.com/user-attachments/assets/34f2a609-c6ed-4bf0-8d5e-551d3d2ca087)
  - After setting up pull through cache, validate the created rule.
  ![Image](https://github.com/user-attachments/assets/b1dc709b-099c-43fc-8005-ebaa1a28bfac)
3. **Pull imagefrom upstream registry and to ECR private registry**
  ![Image](https://github.com/user-attachments/assets/fc414db2-8c53-4e2e-8e3b-47bc4994b068)
  - Pull image from docker hub by aws cli as ECR provided pull commands.
  ![Image](https://github.com/user-attachments/assets/76a489df-b6ea-42ee-9e4d-117260bf0e60)
  
- **Deploy web server on docker containers by using docker compose**:
  _docker compose file_
```
    name: sample-web-docker-containers

    services:
    web-container01:
        image: 851725300440.dkr.ecr.ap-southeast-1.amazonaws.com/docker-hub-pwawslearning/yeasy/simple-web
        hostname: container01
        container_name: web-server01
        environment:
        - INSTANCE=Web Server 1
        networks:
        - simple-network
        
    web-container02:
        image: 851725300440.dkr.ecr.ap-southeast-1.amazonaws.com/docker-hub-pwawslearning/yeasy/simple-web
        hostname: container02
        container_name: web-server02
        environment:
        - INSTANCE=Web Server 2
        networks:
        - simple-network
    
    nginx:
        image: nginx:latest
        container_name: nginx-load-balancer
        ports:
        - "8080:80"
        volumes:
        - ./nginx.conf:/etc/nginx/nginx.conf:ro
        depends_on:
        - web-container01
        - web-container02
        networks:
        - simple-network

    networks:
    simple-network:
        driver: bridge
```
 _nginx.conf_
 ```
    events {}

    http {
        upstream simpleweb {
            server web-container01:80;
            server web-container02:80;
        }

        server {
            listen 80;

            location / {
                proxy_pass http://simpleweb;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
            }
        }
    }
 ```
 ---

## Verification
- Check container network ```docker inspect network bridge```
![Image](https://github.com/user-attachments/assets/831cf3d6-6c88-498d-b92d-67aba884a7b3)
- Access the website by using localhost ```http://localhost:8080```
![Image](https://github.com/user-attachments/assets/28f089bb-2394-446d-a1ce-9e3e2c76ce5b)

![Image](https://github.com/user-attachments/assets/6470c7f8-a628-45a1-ae52-542004af38b3)