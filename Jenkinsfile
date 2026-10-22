
pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    environment {
        DOCKER_IMAGE = "incrdble/java-app:latest"
        KUBECTL = "C:\\Users\\incri\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe"
        KUBECONFIG_PATH = "C:\\Windows\\System32\\config\\systemprofile\\.kube\\config"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/JEYSUDHARSAN/RepoforKuber.git'
            }
        }

        stage('Build with Maven') {
            steps {
                bat 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE% .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dh-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat '''
                        echo %DOCKER_PASS%| docker login -u %DOCKER_USER% --password-stdin
                        if errorlevel 1 exit /b 1

                        docker push %DOCKER_IMAGE%
                        if errorlevel 1 exit /b 1
                    '''
                }
            }
        }

        stage('Refresh Kubernetes Connection') {
            steps {
                bat '''
                    if not exist "%KUBECONFIG_PATH%" (
                        echo Kubernetes configuration file not found.
                        exit /b 1
                    )

                    "%KUBECTL%" --kubeconfig="%KUBECONFIG_PATH%" get nodes
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat '''
                    "%KUBECTL%" --kubeconfig="%KUBECONFIG_PATH%" apply -f deployment.yaml
                    if errorlevel 1 exit /b 1

                    "%KUBECTL%" --kubeconfig="%KUBECONFIG_PATH%" rollout restart deployment/java-app
                    if errorlevel 1 exit /b 1

                    "%KUBECTL%" --kubeconfig="%KUBECONFIG_PATH%" rollout status deployment/java-app --timeout=120s
                    if errorlevel 1 exit /b 1
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}
