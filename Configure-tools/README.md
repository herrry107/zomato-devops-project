# Tools Configuration

**Create SonarQube token**

- SonarQube Console -> Administrator -> Security -> Users -> Click on  3 bars under Token -> Give Name and Generate

![SonarQube-Token](https://github.com/herrry107/zomato-devops-project/blob/main/images/sonarqube-toke-generate.png)

![SonarQube-Token](https://github.com/herrry107/zomato-devops-project/blob/main/images/sonarqube-toke-generate1.png)

**Tools Configuration in Jenkins**

- Jenkins -> Manage Jenkins -> Tools -> JDK installations -> Add Jdk -> jdk17 -> Install automatically -> jdk-17.0.8.1+1

![JDK-Configure](https://github.com/herrry107/zomato-devops-project/blob/main/images/jenkins-tools-jdk.png)

- Jenkins -> Manage Jenkins -> Tools -> Git installations -> Add Git

![Git-Configure](https://github.com/herrry107/zomato-devops-project/blob/main/images/jenkins-tools-git.png)

- Jenkins -> Manage Jenkins -> Tools -> SonarQube Scanner installations -> Add SonarQube Scanner -> sonar-scanner -> Install automatically -> Install from Maven Central

![SonarQube-Configure](https://github.com/herrry107/zomato-devops-project/blob/main/images/jenkins-tools-sonarqube.png)

- Jenkins -> Manage Jenkins -> Tools -> NodeJS installations -> Add NodeJS -> node23 -> Install automatically -> Install from nodejs.org

![NodeJS-Configure](https://github.com/herrry107/zomato-devops-project/blob/main/images/jenkins-tools-nodejs.png)

- Jenkins -> Manage Jenkins -> Tools -> Dependency-Check installations -> Add Dependency-Check -> DP-Check -> Install automatically

![Dependency-Check-Configure](https://github.com/herrry107/zomato-devops-project/blob/main/images/jenkins-tools-DP-Check.png)

- Jenkins -> Manage Jenkins -> Tools -> Docker installations -> Add Docker -> docker -> Install automatically -> Download from docker.com -> latest
- 

**Add SolarQube Credentials in Jenkins**

Manage Jenkins ->  Credentials ->  global-> Add Credentials -> Kind (Secret text) -> secret (token) -> id (any-name)

![SolarQube-Credential](https://github.com/herrry107/zomato-devops-project/blob/main/images/jenkins-sonar-token-secret-credentials.png)

**Add Dockerhub Credentials in Jenkins**

Manage Jenkins ->  Credentials ->  global-> Add Credentials -> Kind (Username with Password) -> Username (dockerhub-id) -> Password (dockerhub-password) -> id (any-name)

![Docker-Credentials](https://github.com/herrry107/zomato-devops-project/blob/main/images/docker-login-credentials.png)

**Add Email to get Notification of Success and Failure**

Go to App Password in Security and generate password and save in jenkins credentials (in jenkins credentials remove app password spaces like if password is "ab cd" than "abcd")

![Gmail-App-Password](https://github.com/herrry107/zomato-devops-project/blob/main/images/gmail-app-password.png)

Manage Jenkins ->  Credentials ->  global-> Add Credentials -> Kind (Username with Password) -> Username (gmail address) -> Password (App-password) -> id (email-creds)

![Gmail-App-Password](https://github.com/herrry107/zomato-devops-project/blob/main/images/docker-sonarqube-both-credentials.png)

# SonarQube Configure in Jenkins/Manage/System

Go to Jenkins -> Manage Jenkins -> System -> SonarQube Servers 

![SonarQube-Server-Configure](https://github.com/herrry107/zomato-devops-project/blob/main/images/sonarqube-configure-in-jenkins-system.png)

# Extended E-mail Notification in Jenkins/Manage/System

Extended E-mail Notification configure for notification 

![Extended E-mail Notification](https://github.com/herrry107/zomato-devops-project/blob/main/images/Extended-email-notification.png)

# E-mail Notification in Jenkins/Manage/System (remove app password spaces)

![E-mail Notification](https://github.com/herrry107/zomato-devops-project/blob/main/images/e-mail-notification1.png)

![E-mail Notification](https://github.com/herrry107/zomato-devops-project/blob/main/images/e-mail-notification2.png)

# Default triggers for email in Jenkins/Manage/System (remove app password spaces)

![E-mail Notification](https://github.com/herrry107/zomato-devops-project/blob/main/images/notification-default-triggers.png)

# Create sonarqube-webhooks

Go to sonarqube -> Administrator -> Configurations  -> Webhooks

![sonarqube-webhooks](https://github.com/herrry107/zomato-devops-project/blob/main/images/sonarqube-webhook1.png)

![sonarqube-webhooks](https://github.com/herrry107/zomato-devops-project/blob/main/images/sonarqube-webhook2.png)

# Build Pipeline

New Item -> Pipeline
<pre><code>
  pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node23'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage ("clean workspace") {
            steps {
                cleanWs()
            }
        }
        stage ("Git Checkout") {
            steps {
                git 'https://github.com/herrry107/Zomato-Jenkins-Docker.git'
            }
        }
        stage("Sonarqube Analysis"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato '''
                }
            }
        }
        stage("Code Quality Gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage("Install NPM Dependencies") {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit -n', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
    }
}
        stage ("Trivy File Scan") {
            steps {
                sh "trivy fs . > trivy.txt"
            }
        }
        stage ("Build Docker Image") {
            steps {
                sh "docker build -t zomato ."
            }
        }
        stage ("Tag & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh "docker tag zomato herrypr107/zomato:latest "
                        sh "docker push herrypr107/zomato:latest "
                    }
                }
            }
        }
        stage('Docker Scout Image') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh 'docker-scout quickview herrypr107/zomato:latest'
                       sh 'docker-scout cves herrypr107/zomato:latest'
                       sh 'docker-scout recommendations herrypr107/zomato:latest'
                   }
                }
            }
        }
        stage ("Deploy to Container") {
            steps {
                sh 'docker run -d --name zomato -p 3000:3000 herrypr107/zomato:latest'
            }
        }
    }
    post {
    always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: """
                <html>
                <body>
                    <div style="background-color: #FFA07A; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">Project: ${env.JOB_NAME}</p>
                    </div>
                    <div style="background-color: #90EE90; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">Build Number: ${env.BUILD_NUMBER}</p>
                    </div>
                    <div style="background-color: #87CEEB; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">URL: ${env.BUILD_URL}</p>
                    </div>
                </body>
                </html>
            """,
            to: 'pratikumar13@gmail.com',
            mimeType: 'text/html',
            attachmentsPattern: 'trivy.txt'
        }
    }
}
</code></pre>

save and apply 

now run command in server
<pre><code>
#add jenkins user into docker  group  
sudo usermod -aG docker jenkins
</code></pre>

<pre><code>
#restart jenkins service
sudo systemctl restart jenkins
</code></pre>

build now
