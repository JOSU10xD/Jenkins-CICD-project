# Jenkins CI/CD Project

A fully automated **CI/CD pipeline** built on **Jenkins** for a Java web application (Maven), with infrastructure provisioning (Terraform), configuration management (Ansible), and application deployment.

---

## 🚀 Overview

This repository contains:

* A **Maven-based Java web app** in `webapp/`
* **Jenkinsfile** defining the pipeline
* **Terraform** configuration to provision servers (`prod.tfvars`)
* **Ansible** playbooks in `ansible-config/` to configure servers
* **Deployment manifests** (`deploy.yaml`) for application rollout
* Supporting scripts in `server/` and documentation in `docs/`

When you push to `main`, Jenkins automatically:

1. **Checks out** your code
2. **Builds & tests** the Java app
3. **Archives** the built artifact
4. **Provisions infrastructure** with Terraform
5. **Configures servers** via Ansible
6. **Deploys** the application

---

## 🛠️ Pipeline Stages

The pipeline is defined in **Jenkinsfile** (declarative syntax):

```groovy
pipeline {
  agent any
  environment {
    APP_NAME   = 'my-webapp'
    ARTIFACT   = 'webapp/target/*.jar'
    TF_DIR     = './'
    ANSIBLE_DIR= 'ansible-config'
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build & Test') {
      steps {
        dir('webapp') {
          sh 'mvn clean package -DskipTests=false'
        }
      }
      post {
        always { junit 'webapp/target/surefire-reports/*.xml' }
      }
    }
    stage('Archive Artifact') {
      steps {
        archiveArtifacts artifacts: "${ARTIFACT}", fingerprint: true
      }
    }
    stage('Provision Infra') {
      steps {
        dir("${TF_DIR}") {
          sh 'terraform init -input=false'
          sh "terraform apply -auto-approve -var-file=prod.tfvars"
        }
      }
    }
    stage('Configure Servers') {
      steps {
        dir("${ANSIBLE_DIR}") {
          sh 'ansible-playbook -i inventory/hosts.ini site.yml'
        }
      }
    }
    stage('Deploy Application') {
      steps {
        sh "ansible-playbook -i ansible-config/inventory/hosts.ini deploy-app.yml \
             -e \"artifact=${env.WORKSPACE}/${ARTIFACT}\""
      }
    }
  }
  post {
    success {
      echo '🎉 Pipeline completed successfully!'
    }
    failure {
      mail to: 'team@example.com',
           subject: "CI/CD Pipeline Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           body: "See Jenkins console output at ${env.BUILD_URL}"
    }
  }
}
```

---

## 🔍 What Happens in Each Stage

1. **Checkout**
   Pulls your Git repository into the Jenkins workspace.

2. **Build & Test**

   * Runs `mvn clean package`
   * Executes unit tests and publishes results to Jenkins.

3. **Archive Artifact**
   Saves the `.jar` in Jenkins so you can download it later.

4. **Provision Infra**

   * Initializes Terraform
   * Applies your AWS/Azure/other cloud resources as specified in `prod.tfvars`

5. **Configure Servers**
   Uses Ansible to:

   * Install Java
   * Create application users
   * Open required firewall ports
   * Any other OS-level setup

6. **Deploy Application**

   * Copies the archived `.jar` to each server
   * Updates systemd or runs the deployment scripts in `deploy.yaml`
   * Restarts the service

---

## 📂 Repo Layout

```bash
.
├── Jenkinsfile             # Declarative pipeline definition
├── ansible-config/         # Ansible playbooks & inventory
│   ├── inventory/          # Hosts list
│   └── site.yml            # Main playbook
├── deploy.yaml             # Kubernetes or other deploy manifest
├── docs/                   # Documentation & architecture diagrams
├── pom.xml                 # Parent POM (if any)
├── prod.tfvars             # Terraform variables for production
├── server/                 # Auxiliary scripts for server bootstrap
└── webapp/                 # Java web application
    ├── pom.xml  
    └── src/  
```

---

## 🔧 Prerequisites

* **Jenkins** (with Pipeline, Git, Ansible, Terraform plugins)
* **Java & Maven** on build agents
* **Terraform CLI** installed on the Jenkins controller/agent
* **Ansible** installed on the Jenkins controller/agent
* Credentials configured in Jenkins for SSH, Git, cloud provider, and email

---

## 🚀 Running the Pipeline

1. **Import this project** into Jenkins as a *Multibranch Pipeline* or *Pipeline* job
2. Point it at your GitHub repo URL
3. Provide credentials and any required environment variables
4. Save and run the job—every push to `main` triggers the full CI/CD workflow

---

## 🤝 Contributing

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/my-pipeline`)
3. Make your changes
4. Update `docs/` with any new steps or requirements
5. Submit a Pull Request

---

## 📜 License

This project is licensed under the **Apache 2.0 License**. See [LICENSE](./LICENSE) for details.
