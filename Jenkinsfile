pipeline {
  agent {
    docker {
      image 'abhishekf5/maven-abhishek-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        //I don't need this stage as I have JenkinsFile in source code repository
        //git branch: 'main', url: 'https://github.com/Briez-b/CICD-setup-java-app-Jenkins-example'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'mvn clean package'
        // mvn clean package uses pim.xml file to get all the dependencies needed for app and install them. And build the application.
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://3.75.241.220:9000"
      }
      steps {
        // Here we use token we saved previously to the secret in Jenkins
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "ebryyau/ultimate-cicd:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile"
        // Created 'docker-credentials' secret in Jenkins.
        REGISTRY_CREDENTIALS = credentials('docker-credentials')
      }
      steps {
        script {
            sh 'docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker-credentials") {
                dockerImage.push()
            }
        }
      }
    }
    //Here I use shell script, but I can use image updater instead.
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "CICD-setup-java-app-manifests"
            GIT_USER_NAME = "Briez-b"
        }
        steps {
            //generated this token github and added as secret 'github' in Jenkins
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    rm -rf CICD-setup-java-app-manifests
                    git clone https://github.com/Briez-b/CICD-setup-java-app-manifests.git
                    cd CICD-setup-java-app-manifests
                    git config user.email "zhenyabrishtenpl@gmail.com"
                    git config user.name "Yauheni Bryshten"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s|\\(ebryyau/ultimate-cicd:\\)[0-9]\\+|\\1${BUILD_NUMBER}|g" deployment.yml
                    git add deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    cd ..
                    rm -rf CICD-setup-java-app-manifests
                '''
            }
        }
    }
  }
}
