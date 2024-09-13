# Create Jenkins Groovy Script for deploying Nextjs ui

We have used the Custom GPT application called [idea-to-app](https://chatgpt.com/g/g-Lo7LWfTPH-idea-to-app) to generate this code.

## Prompt 1
Create Jenkins groovy file for deploying a nextjs ui application. groovy script should have various steps explained below,
  - CI stage will have code build, unit test, sonar code analysis, docker image build, docker login to nexus repo, docker push to nexus. 
  - Code will be pushed into bitbucket, and uses feature/develop branch to trigger the pipeline. CI status will be posted as email in platform@xyz.com group.
  - Create another groovy script to trigger the application deployment to AWS eks namespace.
--
Explain the comprehensive steps for a beginner to get started with groovy scripting with detailed steps from pre-requisites, steps for tool config and configuration

## Response for Prompt 1

### Jenkins Groovy Pipeline Script for Deploying a Next.js UI Application

#### 1. **Groovy Script for CI Pipeline**

This script performs the following steps:
- **Code build** using Bitbucket.
- **Unit testing**.
- **SonarQube analysis**.
- **Docker image build**.
- **Docker login** to Nexus repository.
- **Docker image push** to Nexus.
- **Send email notifications** to `platform@xyz.com`.

```groovy
pipeline {
    agent any

    environment {
        NEXUS_CREDENTIALS = credentials('nexus-docker-credentials')
        NEXUS_URL = 'https://nexus.example.com'
        NEXUS_REPO = 'nextjs-app-repo'
        DOCKER_IMAGE = 'nextjs-ui-app'
        BITBUCKET_CREDENTIALS = credentials('bitbucket-credentials')
        SONARQUBE_URL = 'https://sonarqube.example.com'
        SONARQUBE_TOKEN = credentials('sonarqube-token')
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    git branch: 'feature/develop',
                        url: 'https://bitbucket.org/your-company/nextjs-ui-app.git',
                        credentialsId: 'bitbucket-credentials'
                }
            }
        }

        stage('Install Dependencies & Build') {
            steps {
                script {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Unit Test') {
            steps {
                script {
                    sh 'npm run test'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh "sonar-scanner -Dsonar.projectKey=nextjs-ui -Dsonar.sources=. -Dsonar.host.url=$SONARQUBE_URL -Dsonar.login=$SONARQUBE_TOKEN"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $NEXUS_URL/$NEXUS_REPO/$DOCKER_IMAGE:${env.BUILD_NUMBER} ."
                }
            }
        }

        stage('Docker Login to Nexus') {
            steps {
                script {
                    sh "echo $NEXUS_CREDENTIALS_PSW | docker login -u $NEXUS_CREDENTIALS_USR --password-stdin $NEXUS_URL"
                }
            }
        }

        stage('Push Docker Image to Nexus') {
            steps {
                script {
                    sh "docker push $NEXUS_URL/$NEXUS_REPO/$DOCKER_IMAGE:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Send CI Status Email') {
            steps {
                emailext to: 'platform@xyz.com',
                subject: "Next.js UI CI Pipeline - Build #${env.BUILD_NUMBER}",
                body: "CI pipeline completed. Docker image pushed to Nexus repository. Build Number: ${env.BUILD_NUMBER}."
            }
        }
    }
    post {
        failure {
            emailext to: 'platform@xyz.com',
            subject: "Next.js UI CI Pipeline - Build #${env.BUILD_NUMBER} Failed",
            body: "CI pipeline failed for Next.js UI Application. Check Jenkins for more details."
        }
    }
}
```

#### Explanation:
- **Environment Variables**: These store credentials, URLs, and reusable variables like Docker image names and repositories.
- **Stages**:
  - **Checkout Code**: Pulls code from Bitbucket.
  - **Build**: Runs `npm install` and `npm run build`.
  - **Unit Test**: Executes tests with `npm run test`.
  - **SonarQube**: Analyzes the code using SonarQube.
  - **Docker Build & Push**: Builds and pushes Docker images to Nexus.
  - **Email Notifications**: Sends CI results to the email group.

### 2. **Groovy Script to Trigger Deployment to AWS EKS**

This script triggers the deployment of the Next.js application to an AWS EKS cluster. It assumes you have configured kubectl for EKS access.

```groovy
pipeline {
    agent any

    environment {
        AWS_REGION = 'us-west-2'
        EKS_CLUSTER = 'nextjs-eks-cluster'
        NAMESPACE = 'nextjs-namespace'
        DOCKER_IMAGE = 'nextjs-ui-app'
        NEXUS_URL = 'https://nexus.example.com'
        NEXUS_REPO = 'nextjs-app-repo'
    }

    stages {
        stage('Download Docker Image') {
            steps {
                script {
                    sh "docker pull $NEXUS_URL/$NEXUS_REPO/$DOCKER_IMAGE:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Update Kubernetes Deployment') {
            steps {
                script {
                    sh """
                    kubectl set image deployment/nextjs-ui-app nextjs-ui-app=$NEXUS_URL/$NEXUS_REPO/$DOCKER_IMAGE:${env.BUILD_NUMBER} --namespace=$NAMESPACE
                    kubectl rollout status deployment/nextjs-ui-app --namespace=$NAMESPACE
                    """
                }
            }
        }

        stage('Deployment Success') {
            steps {
                emailext to: 'platform@xyz.com',
                subject: "Next.js UI Deployment - Build #${env.BUILD_NUMBER} Succeeded",
                body: "Deployment to EKS succeeded for Next.js UI Application."
            }
        }
    }

    post {
        failure {
            emailext to: 'platform@xyz.com',
            subject: "Next.js UI Deployment - Build #${env.BUILD_NUMBER} Failed",
            body: "Deployment to AWS EKS failed for Next.js UI Application. Check Jenkins for more details."
        }
    }
}
```

#### Explanation:
- **Environment Variables**: AWS region, EKS cluster name, namespace, and Docker image details.
- **Stages**:
  - **Download Docker Image**: Pulls the Docker image from Nexus.
  - **Update Kubernetes Deployment**: Updates the Kubernetes deployment using the new Docker image.
  - **Deployment Success/Failure**: Sends email notifications based on deployment status.

---

### Getting Started with Groovy Scripting for Jenkins

#### 1. **Pre-Requisites**
   - **Jenkins Installed**: Ensure Jenkins is installed on your system.
   - **Jenkins Plugins**: Install these plugins:
     - Git Plugin
     - Pipeline Plugin
     - SonarQube Plugin
     - Email Extension Plugin
     - Docker Pipeline Plugin
     - Kubernetes Plugin (if deploying to Kubernetes)

   - **Docker**: Docker must be installed and configured on the Jenkins agent.
   - **SonarQube**: SonarQube server configured and accessible.
   - **Nexus Repository**: Docker repository on Nexus configured.

#### 2. **Configure Jenkins**
   - **Credentials**: Set up credentials in Jenkins for Bitbucket, Nexus, AWS, and SonarQube.
     - Go to Jenkins Dashboard → Manage Jenkins → Manage Credentials → Add the following:
       - Bitbucket credentials.
       - Nexus Docker credentials.
       - SonarQube token.
   - **SonarQube Configuration**: Go to Jenkins Dashboard → Manage Jenkins → Configure System → Add SonarQube instance.

#### 3. **Groovy Scripting Steps**
   - **Pipeline Syntax**: Groovy is the scripting language used in Jenkins Pipelines (Declarative or Scripted). This gives the flexibility to define complex CI/CD flows.
   - **Declarative Pipeline**: Easier for beginners. It's structured in blocks, which makes it readable.
   - **Stages**: Each task is placed inside a "stage" block (e.g., `stage('Build')`).

#### 4. **Testing the Pipeline**
   - Push code to the feature/develop branch to trigger the pipeline.
   - Monitor the progress in the Jenkins job console.

---

With these scripts and instructions, you should be able to set up your Jenkins CI/CD pipeline for deploying a Next.js application from Bitbucket, building Docker images, and pushing them to Nexus, and finally deploying them on AWS EKS.

---

**Next Steps**:
1. **Add security scanning** to the pipeline using tools like Trivy or Anchore.
2. **Automate rollback** in case the deployment to AWS fails.
3. **Explore Jenkinsfile libraries** for reusable shared libraries.

Would you like assistance with any of these next steps or further customization of your pipeline?

## Prompt 2
How to enable the plugins in jenkins using automated way? is there way to deploy the plugins in jenkins automatically when spinning up the instance?

## Response for Prompt 2
Yes, there is an automated way to install and enable plugins in Jenkins when spinning up an instance. This can be done by using configuration as code, primarily through:

1. **Jenkins Configuration as Code (JCasC) Plugin**: This allows Jenkins to be configured via a YAML file, including plugin installations.
2. **Preinstalling plugins via the Docker image**: You can predefine a list of plugins to be installed when Jenkins starts using the official Jenkins Docker image or a custom Dockerfile.
3. **Automated Installation via Jenkins CLI or REST API**.

### 1. **Using Jenkins Configuration as Code (JCasC)**

The Jenkins Configuration as Code (JCasC) plugin allows Jenkins to be configured via a YAML file. You can use this to specify which plugins should be installed when Jenkins is started.

#### Steps:

1. **Install the Jenkins Configuration as Code (JCasC) Plugin**:
   You can include this plugin in the list of plugins to be installed via your Dockerfile or manually install it for the first time.

2. **Create a `jenkins.yaml` file**: In this file, you can specify plugins to be installed along with other configurations like credentials, jobs, etc.

Example `jenkins.yaml` to install plugins automatically:
```yaml
jenkins:
  systemMessage: "Jenkins Configured Automatically via JCasC"
  securityRealm: jenkins.security.plugins.UserDatabaseSecurityRealm
  authorizationStrategy: jenkins.model.AuthorizationStrategy$Unsecured
  plugins:
    required:
      - git:4.7.1
      - docker-workflow:1.24
      - sonar:2.13
      - workflow-aggregator:2.6
      - pipeline-stage-view:2.19
      - email-ext:2.83
```

3. **Run Jenkins with this configuration file**: You can mount the configuration file when running Jenkins via Docker or configure it in your Jenkins instance.

Using Docker:
```bash
docker run \
  -v /path/to/jenkins.yaml:/var/jenkins_home/casc_configs/jenkins.yaml \
  -e CASC_JENKINS_CONFIG=/var/jenkins_home/casc_configs/jenkins.yaml \
  jenkins/jenkins:lts
```

The `CASC_JENKINS_CONFIG` environment variable points Jenkins to the configuration file containing plugin and system configurations.

### 2. **Pre-install Plugins via Docker Image**

If you're using Jenkins Docker images, you can pre-install plugins by specifying them in a `plugins.txt` file and building a custom Jenkins image.

#### Steps:

1. **Create a `plugins.txt` file**: This file will list the plugins you want to install.

Example `plugins.txt`:
```
git:4.7.1
docker-workflow:1.24
sonar:2.13
workflow-aggregator:2.6
pipeline-stage-view:2.19
email-ext:2.83
```

2. **Use a Dockerfile to install these plugins**:

Create a `Dockerfile` like this:

```Dockerfile
FROM jenkins/jenkins:lts

# Preinstall Jenkins plugins
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt
```

3. **Build and run your custom Jenkins image**:
```bash
docker build -t custom-jenkins .
docker run -p 8080:8080 custom-jenkins
```

This approach ensures that every time you spin up this Docker container, Jenkins will start with the necessary plugins installed.

### 3. **Automated Installation via Jenkins CLI or REST API**

You can also automate plugin installation using Jenkins CLI or REST API after Jenkins is already running.

#### Install Plugins Using Jenkins CLI:
1. **Download Jenkins CLI**:
   You can download the CLI from your Jenkins instance at `http://<jenkins-url>/jnlpJars/jenkins-cli.jar`.

2. **Install Plugins**:
   Use the following command to install plugins via the CLI:
   ```bash
   java -jar jenkins-cli.jar -s http://localhost:8080/ -auth <user>:<api_token> install-plugin git docker-workflow sonar workflow-aggregator
   ```

3. **Restart Jenkins**:
   After the plugins are installed, you may need to restart Jenkins.
   ```bash
   java -jar jenkins-cli.jar -s http://localhost:8080/ -auth <user>:<api_token> safe-restart
   ```

### 4. **Install Plugins Using Jenkins REST API**

Jenkins provides a REST API that can be used to trigger plugin installations. You can make a `POST` request to the Jenkins API to install plugins programmatically.

#### Example:
Use the following `curl` command to install plugins via the Jenkins REST API:
```bash
curl -X POST -u <user>:<api_token> http://<jenkins-url>/pluginManager/installNecessaryPlugins \
  --data '<jenkins><install plugin="git@4.7.1" /></jenkins>'
```

### Complete Workflow for Automatically Deploying Jenkins with Plugins

To fully automate the installation of Jenkins with plugins and configurations, follow these steps:

1. **Create a Custom Docker Image**:
   - Use a `Dockerfile` to pre-install the necessary plugins from a `plugins.txt` file.

2. **Use Jenkins Configuration as Code**:
   - Create a `jenkins.yaml` configuration file to configure Jenkins settings, jobs, credentials, and more.

3. **Automate Deployment Using Infrastructure as Code (IaC)**:
   - Use tools like **Terraform**, **Ansible**, or **Kubernetes** manifests to automate the deployment of Jenkins instances. You can define your Docker image and mount the necessary configuration files.

Here is a basic **Terraform configuration** example to deploy Jenkins on AWS (with Docker image and mounted configurations):

```hcl
resource "aws_instance" "jenkins" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
  key_name      = "my-key-pair"
  
  user_data = <<-EOF
    #!/bin/bash
    docker run -d -p 8080:8080 -p 50000:50000 \
      -v /var/jenkins_home:/var/jenkins_home \
      -v /path/to/jenkins.yaml:/var/jenkins_home/casc_configs/jenkins.yaml \
      -e CASC_JENKINS_CONFIG=/var/jenkins_home/casc_configs/jenkins.yaml \
      custom-jenkins
  EOF

  tags = {
    Name = "Jenkins"
  }
}
```

This Terraform code will launch an EC2 instance and run the Jenkins container with your custom Docker image and JCasC configurations.

---

### Conclusion

To automate the deployment and plugin installation for Jenkins, you can leverage:
- **Jenkins Configuration as Code (JCasC)** for managing Jenkins configurations.
- **Custom Docker Images** with preinstalled plugins.
- **Jenkins CLI or REST API** for programmatic plugin installation.

This approach ensures that Jenkins is configured automatically when spun up, which helps in automating CI/CD pipelines in cloud environments or other automated setups.

## Prompt 3
How to use Terraform, and Kubernetes manifest for deploying Jenkins with these configurations? 

## Response for Prompt 3
Let's walk through how to use **Terraform** and **Kubernetes manifests** to deploy Jenkins with automated plugin installation and configuration.

### 1. **Deploying Jenkins with Terraform**

Terraform can be used to automate the provisioning of infrastructure, including instances that run Jenkins. We'll focus on deploying Jenkins on AWS EC2 using Terraform, and Docker will be used to run Jenkins with Configuration as Code (JCasC).

#### Steps to deploy Jenkins on AWS using Terraform:

1. **Terraform Configuration**:

   Create a `main.tf` file for your Terraform configuration.

```hcl
provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "jenkins" {
  ami           = "ami-12345678" # Replace with the latest Ubuntu/Docker AMI ID
  instance_type = "t2.micro"
  key_name      = "my-key-pair"

  tags = {
    Name = "Jenkins-Terraform"
  }

  user_data = <<-EOF
    #!/bin/bash
    sudo apt-get update -y
    sudo apt-get install -y docker.io
    sudo systemctl start docker
    sudo systemctl enable docker
    sudo docker run -d -p 8080:8080 -p 50000:50000 \
      -v /var/jenkins_home:/var/jenkins_home \
      -v /home/ubuntu/jenkins.yaml:/var/jenkins_home/casc_configs/jenkins.yaml \
      -e CASC_JENKINS_CONFIG=/var/jenkins_home/casc_configs/jenkins.yaml \
      jenkins/jenkins:lts
  EOF

  # EC2 security group to allow HTTP and Jenkins traffic
  vpc_security_group_ids = [aws_security_group.jenkins_sg.id]

  # Use an SSH key pair for remote access (for debugging if needed)
  key_name = var.aws_key_name
}

resource "aws_security_group" "jenkins_sg" {
  name        = "jenkins_sg"
  description = "Allow Jenkins traffic"

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 50000
    to_port     = 50000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Outputs for connecting to Jenkins
output "jenkins_url" {
  value = "http://${aws_instance.jenkins.public_ip}:8080"
}

output "jenkins_public_ip" {
  value = aws_instance.jenkins.public_ip
}
```

#### Steps Explained:
- **User Data Script**: This installs Docker, pulls the Jenkins Docker image, and runs it. We also mount a JCasC configuration file (`jenkins.yaml`) and specify the environment variable `CASC_JENKINS_CONFIG` to enable Configuration as Code for Jenkins.
- **Security Group**: The security group allows HTTP traffic on port 8080 for Jenkins, SSH on port 22, and agent communication on port 50000.

2. **Create a `jenkins.yaml` configuration file**:
   - This file defines your Jenkins configuration (plugins, users, jobs, etc.).

Example `jenkins.yaml`:

```yaml
jenkins:
  systemMessage: "Jenkins Configured Automatically via JCasC"
  securityRealm: jenkins.security.plugins.UserDatabaseSecurityRealm
  authorizationStrategy: jenkins.model.AuthorizationStrategy$Unsecured
  plugins:
    required:
      - git:4.7.1
      - docker-workflow:1.24
      - sonar:2.13
      - workflow-aggregator:2.6
      - pipeline-stage-view:2.19
      - email-ext:2.83
```

Make sure to upload this file on the EC2 instance in `/home/ubuntu/jenkins.yaml`.

3. **Initialize and Apply Terraform**:
   - Run the following commands to apply the configuration:
   ```bash
   terraform init
   terraform apply
   ```

   After Terraform finishes, you’ll see the Jenkins URL output to access your Jenkins instance.

---

### 2. **Deploying Jenkins on Kubernetes with Manifests**

You can also deploy Jenkins using **Kubernetes**. The steps involve defining a `Deployment` for Jenkins, attaching persistent storage, and mounting the configuration files. Here’s how you can do it:

#### Kubernetes Manifest for Jenkins Deployment

1. **Jenkins Deployment YAML**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  labels:
    app: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins
        image: jenkins/jenkins:lts
        ports:
        - containerPort: 8080
        - containerPort: 50000
        env:
        - name: CASC_JENKINS_CONFIG
          value: /var/jenkins_home/casc_configs/jenkins.yaml
        volumeMounts:
        - name: jenkins-home
          mountPath: /var/jenkins_home
      volumes:
      - name: jenkins-home
        persistentVolumeClaim:
          claimName: jenkins-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
  labels:
    app: jenkins
spec:
  ports:
  - port: 8080
    targetPort: 8080
  - port: 50000
    targetPort: 50000
  selector:
    app: jenkins
  type: LoadBalancer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

#### Explanation:
- **Deployment**:
  - Jenkins runs as a container.
  - We specify `CASC_JENKINS_CONFIG` to point to the configuration file (`jenkins.yaml`) for Jenkins Configuration as Code.
  - Persistent volume claim (PVC) is used for data persistence (`jenkins_home` directory).
- **Service**:
  - Exposes Jenkins on ports `8080` and `50000`. A `LoadBalancer` service type is used, which is common in cloud-based Kubernetes deployments.

2. **Persistent Volume**:
   Ensure you have a persistent volume claim (PVC) for Jenkins data. The `jenkins-pvc` in the manifest will claim storage for Jenkins' persistent data.

3. **Create a ConfigMap for JCasC** (Optional):
   You can use a ConfigMap to store your Jenkins YAML configuration for JCasC, and mount it into the container.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: jenkins-casc-config
data:
  jenkins.yaml: |
    jenkins:
      systemMessage: "Jenkins Configured Automatically via JCasC"
      securityRealm: jenkins.security.plugins.UserDatabaseSecurityRealm
      authorizationStrategy: jenkins.model.AuthorizationStrategy$Unsecured
      plugins:
        required:
          - git:4.7.1
          - docker-workflow:1.24
          - sonar:2.13
          - workflow-aggregator:2.6
          - pipeline-stage-view:2.19
          - email-ext:2.83
```

Mount this ConfigMap as a volume in the `Deployment` like this:

```yaml
volumeMounts:
- name: jenkins-casc
  mountPath: /var/jenkins_home/casc_configs/jenkins.yaml
  subPath: jenkins.yaml

volumes:
- name: jenkins-casc
  configMap:
    name: jenkins-casc-config
```

4. **Deploy to Kubernetes**:
   Apply the Kubernetes manifests with:
   ```bash
   kubectl apply -f jenkins-deployment.yaml
   ```

5. **Access Jenkins**:
   Once the service is created, Kubernetes will provide you with a LoadBalancer IP or DNS to access the Jenkins UI.

---

### Conclusion

- **Terraform**: Automates the deployment of Jenkins on AWS EC2, with a Docker-based Jenkins instance using Jenkins Configuration as Code (JCasC).
- **Kubernetes**: Uses `Deployment`, `Service`, and `PersistentVolumeClaim` to deploy Jenkins on a Kubernetes cluster, mounting the JCasC configuration for plugin and system setup.

These configurations provide a robust and automated way to deploy Jenkins with plugins and configurations pre-installed.