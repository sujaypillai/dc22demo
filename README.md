# DockerCon22 Demo

# Running containers locally
```
docker compose up
```

# Running containers on Cloud using Docker Desktop

- [Docker and ACI](https://docs.docker.com/engine/context/aci-integration/)
- [Docker and ECS](https://docs.docker.com/engine/context/ecs-integration/)


# Azure CLI Command to create an ACI instance
```
az container create \
   --resource-group rg-dockeraci \
   --name dockercon22aci
   --image sujaypillai/simplewhale 
   --dns-name-label acidockerdemo 
   --ports 80
```

## 1. Login to Azure
```
docker login azure
```
## 2. Create an ACI Context
```
docker context create aci azdockercon22
```

### 3. Run your compose app
```
docker compose up
```

# Amazon ECS deployment

### Create AWS Context
```
docker context create ecs awsdockercon22
```

```
docker context ls
docker context ls
NAME                TYPE                DESCRIPTION                               DOCKER ENDPOINT               KUBERNETES ENDPOINT   ORCHESTRATOR                                                                            
azcloud             aci                 rg-dockeraci@southeastasia                                                                    
default             moby                Current DOCKER_HOST based configuration   unix:///var/run/docker.sock                         swarm
dockerpenangdemo *  ecs                 ap-southeast-1               
```

### Accessing Private Docker images
To pull private images from another registry, including Docker Hub, you’ll have to create a Username + Password (or a Username + Token) secret on the [AWS Secrets Manager service](https://docs.aws.amazon.com/secretsmanager/).

```
docker secret create hubaccesstoken --username sujaypillai --password <yourDockerHubToken>
arn:aws:secretsmanager:ap-southeast-1:123456789101:secret:hubaccesstoken-asrWLR
```

```
docker secret ls            
ID                                                                                NAME                DESCRIPTION
arn:aws:secretsmanager:ap-southeast-1:123456789101:secret:hubaccesstoken-asrWLR   hubaccesstoken  
```

Once created, you can use this ARN in your Compose file using using `x-aws-pull_credentials` custom extension with the Docker image URI for your service.
```
version: 3.8
services:
  worker:
    image: sujaypillai/privateimage
    x-aws-pull_credentials: "arn:aws:secretsmanager:ap-southeast-1:123456789101:secret:hubaccesstoken-asrWLR"
```

### Login to Amazon ECR
`aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 123456789101.dkr.ecr.ap-southeast-1.amazonaws.com`

### Pushing image to Amazon ECR
Tag the locally developed image to in the form `aws_account_id.dkr.ecr.region.amazonaws.com/repositoryname`

```docker tag sujaypillai/timestamper 123456789101.dkr.ecr.ap-southeast-1.amazonaws.com/timestamper:1.0```

```docker push 123456789101.dkr.ecr.ap-southeast-1.amazonaws.com/timestamper:1.0```

### Running your docker container on Amazon ECS

| Command                   | Description |
|---------------------------|-------------|
| ``` docker compose up ```                         |   This by default uses `docker-compose.yml`           |
| ```docker compose up -f docker-compose-ecs.yml``` |   Specifying a particular compose file          |


### Check the status for your services running on Amazon ECS
```docker compose ps```

### Convert your existing docker-compose definition to CloudFormation template
```docker compose convert > cf-template.yml```

### Stream the logs on to your terminal
```docker compose logs```

### Tear down the cluster created above 
```docker compose down```
