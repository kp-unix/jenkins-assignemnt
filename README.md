**Jenkins CI/CD Pipeline Project – Assignment** 


**Prerequisites**

Jenkins running on an EC2 instance
Git and GitHub account
WSL (Windows Subsystem for Linux) for development
Java (OpenJDK 17)
Maven
JUnit for testing
SSH access between Jenkins and EC2 staging instance


**Jenkins Installation on EC2**

**Install Java and Jenkins:**
sudo apt update
sudo apt install openjdk-17-jdk -y
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y
sudo systemctl restart jenkins

Jenkins URL:
http://localhost:8080

**Connecting Jenkins with GitHub**
Install Git plugin in Jenkins
Create a GitHub Personal Access Token (PAT)
Add the PAT in Jenkins credentials (Secret Text)
In the Jenkins job, set the Git repository URL
Use the credential you added

**setting Webhook for auto trigger.**
 Settings → Webhooks → Add Webhook


**Add the dependency in pom.xml:**

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>

****Deployment (Example)****

pipeline {
    agent any

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/kalpesh-dev/sample-ci-cd-project.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Deploy to Server') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                sh '''
                echo "Deploying build to remote server..."
                scp -o StrictHostKeyChecking=no target/myapp.jar ubuntu@3.110.92.77:/opt/app/
                ssh -o StrictHostKeyChecking=no ubuntu@3.110.92.77 "sudo systemctl restart myapp"
                '''
            }
        }
    }

    post {
        success {
            echo 'Build and Deployment Successful! - Kalpesh'
        }
        failure {
            echo 'Build Failed. Check logs.'
        }
    }
}


 


