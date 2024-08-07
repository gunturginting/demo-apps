pipeline {
    agent any
    
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment  {
        APP_NAME = "demo-apps"
        DOCKER_USER = "aey16"
        DOCKER_PASS = 'docker-cred'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${env.GIT_COMMIT.substring(0,7)}"
        REPO_CODE = "https://github.com/gunturginting/demo-apps.git"
        REPO_SECRET = "github"
        IMAGE_REPO = "aey16/${APP_NAME}"
        NAMESPACE = "app"
    }
    
    stages {
        stage('SCM') {
            steps {
                git credentialsId: 'github', url: 'https://github.com/gunturginting/demo-apps.git'
            }
        }
        
        stage('Build Jar') {
            steps {
                sh "mvn clean install"
            }
        }
        
        stage('Build Image') {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                        docker_image.push("${IMAGE_TAG}")
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        sh "mkdir -p ~/.kube/"
                        sh "cat ${KUBECONFIG} >> ~/.kube/config"
                        
                        sh """
                        helm upgrade --install ${APP_NAME} ./config/${APP_NAME} \
                        --set-string image.repository=${IMAGE_REPO},image.tag=${IMAGE_TAG} \
                        -f ./config/${APP_NAME}/values.yaml --debug --namespace ${NAMESPACE}
                        """
                    }
                }
            }
        }
        
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
    }
}

