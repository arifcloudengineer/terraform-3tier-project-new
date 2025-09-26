Production Grade 3-Tier Architecture

This project contains a **complete production-grade 3-tier architecture** with the following components:
- Web Tier  
- App Tier  
- DB Tier  
- Security Groups (SGs)  
- Auto Scaling Groups (ASG)  
- Elastic Load Balancer (ELB)  

---

## 🚀 Setup Instructions

### 1. Launch EC2 Instance
- Launch an **Amazon Linux 2023 instance**.  
- Install **Jenkins** and **Terraform** on the same machine:

```bash
sh Jenkins.sh
sh terraform.sh
```

> **Note:**  
> - `terraform.sh` contains `aws configure`.  
> - Configure AWS **KEYS** here (if not using IAM Roles).  

---

### 2. Create IAM Role
- Create an **IAM Role** with **AdministratorAccess**.  
- Attach the role to the EC2 instance running Jenkins.  

---

### 3. Access Jenkins
- Open Jenkins in your browser:  
  ```
  http://<EC2-Public-IP>:8080
  ```

- Install required plugins:
  - **Pipeline: Stage View Plugin**  
  - **Terraform Plugin**  

---

### 4. Create Pipeline in Jenkins
- Mark the project as **parameterized** with:
  - **Name:** `action`  
  - **Choices:** `apply`, `destroy`  

---

## 📂 Terraform Backend Configuration

Before running the pipeline, create:  
- **S3 Bucket** (for storing remote state)  
- **DynamoDB Table** (for state locking, column: `LockID`)  

```hcl
terraform {
  backend "s3" {
    bucket         = "reya-tfstate-bucket-123"
    key            = "3tier/prod/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "dynamodb-terraform-state-lock"
    encrypt        = true
  }
}
```

---

## 🛠️ Jenkins Pipeline Script

```groovy
pipeline {
  agent any
  stages {
    stage('checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/ReyazShaik/terraform-3tier-project-new.git'
      }
    }
    stage('init') {
      steps {
        sh 'terraform init'
      }
    }
    stage('validate') {
      steps {
        sh 'terraform validate'
      }
    }
    stage('plan') {
      steps {
        sh 'terraform plan'
      }
    }
    stage('action') {
      steps {
        sh 'terraform $action --auto-approve'
      }
    }
  }
}
```

---

## 🌐 Accessing the Infrastructure
- Once Terraform applies successfully, copy the **ELB DNS Name** from the outputs.  
- Paste it in the browser to access your application.  

---

## 🔍 Useful Terraform Commands

```bash
# List all resources in state
terraform state list

# Count number of resources
terraform state list | wc -l

# Generate infrastructure graph
terraform graph
```

👉 Use [Graphviz Online](https://dreampuf.github.io/GraphvizOnline/) to visualize the graph:
1. Copy output of `terraform graph`  
2. Paste it into Graphviz Online  
3. Download the infrastructure blueprint image  

---

## 📌 Notes
- **Project 2 Repo:** `terraform-3tier-project-new`  
- Supports **Apply** and **Destroy** actions via Jenkins parameterized pipeline.  
