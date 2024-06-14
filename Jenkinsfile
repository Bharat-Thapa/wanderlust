pipeline{
    agent {label "Dev-Server}"
    environment{
        SONAR_HOME= tool "Sonar"
    }
    stages{
        stage("Clone Code from GitHub"){
            steps{
                git url: "https://github.com/krishnaacharyaa/wanderlust.git", branch: "main"
                echo "Code successfully cloned"
            }
        }
        stage("SonarQube Quality Analysis"){
            steps{
                withSonarQubeEnv("Sonar"){
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=wanderlust -Dsonar.projectKey=wanderlust"
                }
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Sonar Quality Gate Scan"){
            steps{
                timeout(time: 2, unit: "MINUTES"){
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage("Trivy File System Scan"){
            steps{
                sh "trivy fs --format  table -o trivy-fs-report.html ."
            }
        }
        stage("build and test"){
            steps{
                sh "docker build -t wanderlust-app ."
                echo "code built and tested successfully"
            }
        }

        stage("Push"){
            steps {
                echo "Pushing the image to docker hub"
                withCredentials([usernamePassword(credentialsId:"dockerHub",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
                sh "docker tag wanderlust-app ${env.dockerHubUser}/wanderlust-app:latest"
                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                sh "docker push ${env.dockerHubUser}/wanderlust-app:latest"
                }
            }
        }

       stage('Deploy') {
    steps {
        withCredentials([file(credentialsId: KUBECONFIG_CREDENTIALS_ID, variable: 'KUBECONFIG')]) {
            sh 'helm upgrade --install wanderlust-app ./helm-chart --set image.repository=${env.dockerHubUser}/reddit-clone-app --set image.tag=latest --kubeconfig $KUBECONFIG'
        }
    }
}


    post {
        always {
            cleanWs()
        }
    }
}
