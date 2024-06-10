### Infrastructure as Code (IaC) with Terraform for AWS ECS/Fargate

Here’s a step-by-step guide to set up a "Hello World" Node.js app on AWS ECS using Fargate with Terraform, along with a CI/CD pipeline using GitHub Actions.

#### Step 1: Set up the Node.js Application
1. **Create a basic Node.js app:**
2. **Add dependencies:**

#### Step 2: Dockerize the Application
1. **Create a Dockerfile:**

#### Step 3: Create Terraform Configuration
1. **Set up Terraform configuration files:**

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
