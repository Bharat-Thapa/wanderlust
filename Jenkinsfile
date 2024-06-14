pipeline {
    agent { label 'Dev-Server' }
    environment {
        SONAR_HOME = tool 'Sonar'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        KUBECONFIG_CREDENTIALS_ID = 'kubeconfig'
    }
    stages {
        stage('Clone Code from GitHub') {
            steps {
                git url: 'https://github.com/krishnaacharyaa/wanderlust.git', branch: 'main'
                echo 'Code successfully cloned'
            }
        }
        stage('SonarQube Quality Analysis') {
            steps {
                withSonarQubeEnv('Sonar') {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=wanderlust -Dsonar.projectKey=wanderlust"
                }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Sonar Quality Gate Scan') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage('Trivy File System Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage('Build and Test Frontend') {
            steps {
                dir('frontend') {
                    sh 'docker build -t wanderlust-frontend .'
                }
                echo 'Frontend code built and tested successfully'
            }
        }
        stage('Build and Test Backend') {
            steps {
                dir('backend') {
                    sh 'docker build -t wanderlust-backend .'
                }
                echo 'Backend code built and tested successfully'
            }
        }
        stage('Push Frontend Image') {
            steps {
                echo 'Pushing the frontend image to Docker Hub'
                withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPass', usernameVariable: 'dockerHubUser')]) {
                    sh 'docker tag wanderlust-frontend ${env.dockerHubUser}/wanderlust-frontend:latest'
                    sh 'docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}'
                    sh 'docker push ${env.dockerHubUser}/wanderlust-frontend:latest'
                }
            }
        }
        stage('Push Backend Image') {
            steps {
                echo 'Pushing the backend image to Docker Hub'
                withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPass', usernameVariable: 'dockerHubUser')]) {
                    sh 'docker tag wanderlust-backend ${env.dockerHubUser}/wanderlust-backend:latest'
                    sh 'docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}'
                    sh 'docker push ${env.dockerHubUser}/wanderlust-backend:latest'
                }
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([file(credentialsId: 'KUBECONFIG_CREDENTIALS_ID', variable: 'KUBECONFIG')]) {
                    sh '''
                    helm upgrade --install wanderlust-frontend ./helm-chart/frontend --set image.repository=${env.dockerHubUser}/wanderlust-frontend --set image.tag=latest --kubeconfig $KUBECONFIG
                    helm upgrade --install wanderlust-backend ./helm-chart/backend --set image.repository=${env.dockerHubUser}/wanderlust-backend --set image.tag=latest --kubeconfig $KUBECONFIG
                    '''
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
