### Infrastructure as Code (IaC) with Terraform for AWS ECS/Fargate

Here’s a step-by-step guide to set up a "Hello World" Node.js app on AWS ECS using Fargate with Terraform, along with a CI/CD pipeline using GitHub Actions.

#### Step 1: Set up the Node.js Application

1. **Create a basic Node.js app:**
   ```javascript
   // app.js
   const express = require('express');
   const app = express();
   const port = 3000;

   app.get('/', (req, res) => {
       res.send('Hello World!');
   });

   app.listen(port, () => {
       console.log(`App running on http://localhost:${port}`);
   });
   ```

2. **Add dependencies:**
   ```json
   // package.json
   {
     "name": "hello-world-app",
     "version": "1.0.0",
     "description": "Hello World Node.js App",
     "main": "app.js",
     "scripts": {
       "start": "node app.js"
     },
     "dependencies": {
       "express": "^4.17.1"
     }
   }
   ```

#### Step 2: Dockerize the Application

1. **Create a Dockerfile:**
   ```dockerfile
   FROM node:14

   WORKDIR /app

   COPY package*.json ./

   RUN npm install

   COPY . .

   EXPOSE 3000

   CMD ["npm", "start"]
   ```

#### Step 3: Create Terraform Configuration

1. **Set up Terraform configuration files:**

   - **main.tf:**
     ```hcl
     provider "aws" {
       region = "us-east-1"
     }

     resource "aws_ecs_cluster" "hello_world_cluster" {
       name = "hello-world-cluster"
     }

     resource "aws_iam_role" "ecs_task_execution_role" {
       name = "ecsTaskExecutionRole"
       assume_role_policy = jsonencode({
         Version = "2012-10-17",
         Statement = [{
           Effect = "Allow",
           Principal = {
             Service = "ecs-tasks.amazonaws.com"
           },
           Action = "sts:AssumeRole"
         }]
       })

       managed_policy_arns = [
         "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
       ]
     }

     resource "aws_ecs_task_definition" "hello_world_task" {
       family                   = "hello-world-task"
       network_mode             = "awsvpc"
       requires_compatibilities = ["FARGATE"]
       cpu                      = "256"
       memory                   = "512"

       execution_role_arn = aws_iam_role.ecs_task_execution_role.arn

       container_definitions = jsonencode([{
         name = "hello-world-container"
         image = "your-dockerhub-username/hello-world-app:latest"
         essential = true
         portMappings = [{
           containerPort = 3000
           hostPort      = 3000
         }]
       }])
     }

     resource "aws_ecs_service" "hello_world_service" {
       name            = "hello-world-service"
       cluster         = aws_ecs_cluster.hello_world_cluster.id
       task_definition = aws_ecs_task_definition.hello_world_task.arn
       desired_count   = 1

       launch_type = "FARGATE"

       network_configuration {
         subnets         = ["your-subnet-id"]
         security_groups = ["your-security-group-id"]
       }
     }
     ```

#### Step 4: Set up GitHub Actions for CI/CD

1. **Create a GitHub Actions workflow file:**
   - **.github/workflows/deploy.yml:**
     ```yaml
     name: CI/CD Pipeline

     on:
       push:
         branches:
           - main

     jobs:
       build:
         runs-on: ubuntu-latest

         steps:
         - name: Checkout code
           uses: actions/checkout@v2

         - name: Set up Node.js
           uses: actions/setup-node@v2
           with:
             node-version: '14'

         - name: Install dependencies
           run: npm install

         - name: Run tests
           run: npm test

         - name: Build Docker image
           run: docker build -t your-dockerhub-username/hello-world-app:latest .

         - name: Log in to DockerHub
           run: echo "${{ secrets.DOCKERHUB_PASSWORD }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin

         - name: Push Docker image
           run: docker push your-dockerhub-username/hello-world-app:latest

         - name: Terraform init
           run: terraform init
           working-directory: ./terraform

         - name: Terraform apply
           run: terraform apply -auto-approve
           working-directory: ./terraform
           env:
             AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
             AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
     ```

#### Step 5: GitHub Repository and Screencast

1. **Create a GitHub repository:**
   - [Create a new repository](https://github.com/new) and push your code.

2. **Secrets in GitHub:**
   - Go to the repository settings and add the following secrets:
     - `DOCKERHUB_USERNAME`
     - `DOCKERHUB_PASSWORD`
     - `AWS_ACCESS_KEY_ID`
     - `AWS_SECRET_ACCESS_KEY`

3. **Push your code to GitHub:**
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/your-username/your-repository.git
   git push -u origin main
   ```
