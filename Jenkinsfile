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
        
        stage('Checkout GitOps Repo to update manifests') {
         steps {  //dir() tells Jenkins to run the steps inside a specific folder in the workspace.Switch to manifest folder to clone manifest repos & to update files 
            dir('manifests') { 
               git url: 'https://github.com/srinathsidhu12/Spring_boot_app_argocd_CD.git'
            }
         }
        }  
        stage('Update k8s manifests') {
            steps { 
               dir('manifests') { 
                 withCredentials([usernamePassword(
                   credentialsId: 'github-creds',
                   usernameVariable: 'GIT_USER',
                   passwordVariable: 'GIT_PASS'
             )]) {

             sh """          
                sed -i 's|image:.*|image: ${DOCKER_HUB_REPO}:${IMAGE_TAG}|' deployment.yaml

                git commit -am "Update image to ${DOCKER_HUB_REPO}:${IMAGE_TAG}"

                git push origin master
             """   
             }
          } 
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

