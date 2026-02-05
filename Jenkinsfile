pipeline {
    agent any
    environment {
        DOCKER_HUB_REPO = "srinathsidhu12/springboot-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    tools {
       jdk 'JDK-21'
       maven 'Maven-3.9.11'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/srinathsidhu12/Spring_boot_app_Jenkins_CI.git'	
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                  docker build -t ${DOCKER_HUB_REPO}:${IMAGE_TAG} .
                """
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-id',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                      echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                      docker push ${DOCKER_HUB_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update k8s manifests') {
            steps {
               git credentialsId: 'github-creds',
                   url: 'https://github.com/user/k8s-manifests.git'

             sh """
                sed -i 's|image:.*|image: ${DOCKER_HUB_REPO}:${IMAGE_TAG}|' deployment.yaml
                git commit -am "Update image to ${DOCKER_HUB_REPO}:${IMAGE_TAG}"
                git push origin master
             """   
             }
         }    
    }

    post {
        success {
            echo "CI pipeline completed &  K8s manifests updated successfully"
        }
        failure {
            echo "CI failed!"
        }
    }
}

