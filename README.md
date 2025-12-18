# Student App - CI/CD Pipeline Using Jenkins & Tomcat10

> This repository contains a **java Maven-based Student Application** packaged as a **War** and deployed automatically to **Apache tomcat10** using **Jenkins CI/CD Pipeline** .
This Project demonstrates a complete **DevOps Workflow** including source control , build automation , and remote deployment on an Ubuntu server .

---
## Prerequisites 

### 1. Infrastructure 
- **Jenkins server** -> runs jenkins pipeline 
- **Student-app server** -> hosts Tomcat10 for deployment
- Both launched with the same ssh-key (.pem)

### 2. Software Requirements 
| Component         | Jenkins Server | Student-App Server |
| ----------------- | -------------- | ------------------ |
| Java (OpenJDK 17) | ✅ Required     | ✅ Required         |
| Git               | ✅ Required     | ✅ Required         |
| Jenkins           | ✅ Required     | ❌ Not needed       |
| Maven             | ✅ Required     | ❌ Not needed       |
| Tomcat10          | ❌ Not needed   | ✅ Required         |

---

## Installation 

### On Jenkins Server

#### Step 1 : Install java 

bash 

     sudo apt install -y openjdk-17-jdk

#### Step 2 : Install Maven

bash
   
    sudo apt install -y maven
    mvn --version

![my images](./images/Screenshot%202025-12-18%20130230.png)

### On Student-app Server

#### Step 1 : Install java 
bash
    
    sudo apt install -y openjdk-17-jdk

#### Step 2 : Install Tomcat10

bash 

    sudo apt install -y tomcat10
    sudo systemctl enable tomcat10
    sudo systemctl start tomcat10

![my images](./images/Screenshot%202025-12-18%20130420.png)

![my images](./images/Screenshot%202025-12-16%20110650.png)
---

## Jenkins Setup 

### Step 1 : Add SSH Credentials 
- Go to manage jenkins -> credentials -> (global) -> Add credentials 
- Kind : SSH username with private key
- Username : `Ubuntu` 
- Private key : Paste your `.pem` file contents 
- ID : `student-app-key`

![my images](./images/Screenshot%202025-12-18%20130619.png)
### Step 2 : Create Pipeline Job

- Go to Jenkins -> New item -> pipeline -> create 
- Under pipeline defination -> choose pipeline script from SCM 
- SCM : git 
- Repo url : `https://github.com/Nikhil-patil678/Student-app.git`
- Branch : `Main`

![my images](./images/Screenshot%202025-12-18%20130718.png)
---

## Jenkinsfile 

bash
   
    pipeline {
    agent any

    environment {
        SERVER_IP    = '100.31.91.196'
        SSH_CRED_ID  = 'student-app-key'
        TOMCAT_PATH  = '/var/lib/tomcat10/webapps'
        TOMCAT_SVC   = 'tomcat10'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Nikhil-patil678/Student-app.git'
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sshagent([SSH_CRED_ID]) {
                    sh """
                        WAR_FILE=\$(ls target/*.war | head -n 1)
                        scp -o StrictHostKeyChecking=no \$WAR_FILE ubuntu@${SERVER_IP}:/tmp/
                        ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} '
                            sudo rm -rf ${TOMCAT_PATH}/*
                            sudo mv /tmp/*.war ${TOMCAT_PATH}/ROOT.war
                            sudo chown tomcat:tomcat ${TOMCAT_PATH}/ROOT.war
                            sudo systemctl restart ${TOMCAT_SVC}
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "App deployed! Visit: http://${SERVER_IP}:8080/"
        }
        failure {
            echo "Deployment failed."
        }
    }
    }

![my images](./images/Screenshot%202025-12-18%20130827.png)
---

### Access Your App 
Visit  your deployed application at :

bash 
    
    http://<Student-app-SERVER-IP>:8080/

This setup demonstrates a complete CI/CD pipeline for Java applications using Maven and Jenkins.
---