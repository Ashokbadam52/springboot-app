pipeline {
    agent any

    environment {
        IMAGE = 'yourdockerhubusername/springboot-app'
        SONARQUBE = 'SonarQube' // Jenkins global config
    }

    tools {
        maven 'Maven 3'
        jdk 'JDK 17'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/yourusername/springboot-app.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=springboot-app -Dsonar.login=$SONAR_AUTH_TOKEN'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t $IMAGE:${BUILD_NUMBER} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push $IMAGE:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                  sed -i 's|image: .*|image: $IMAGE:${BUILD_NUMBER}|' k8s-deployment.yaml
                  kubectl apply -f k8s-deployment.yaml
                """
            }
        }
    }

    post {
        success {
            echo '✅ Build and Deploy Successful!'
        }
        failure {
            echo '❌ Build Failed!'
        }
    }
}
