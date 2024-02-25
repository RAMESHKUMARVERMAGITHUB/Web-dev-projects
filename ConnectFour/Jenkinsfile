pipeline{
    agent any
    // tools{
    //     jdk 'jdk17'
    //     nodejs 'node16'
    // }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/rameshkumarvermagithub/Web-dev-projects.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
              dir('ConnectFour') {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=connectfour \
                    -Dsonar.projectKey=connectfour'''
                }
              }
            }
        }
        stage("quality gate"){
           steps {
                script {
                  dir('ConnectFour') {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
                }
                }
            }
        }
        // stage('Install Dependencies') {
        //     steps {
        //         sh "npm install"
        //     }
        // }
        stage('OWASP FS SCAN') {
            steps {
              dir('ConnectFour') {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
            }
        }
         stage('TRIVY FS SCAN') {
            steps {
              dir('ConnectFour') {
                sh "trivy fs . > trivyfs.txt"
            }
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                  dir('ConnectFour') {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t rameshkumarverma/connectfour ."
                       // sh "docker tag connectfour rameshkumarverma/connectfour:latest"
                       sh "docker push rameshkumarverma/connectfour:latest"
                    }
                  }
                }
            }
        }
        stage("TRIVY"){
            steps{
              dir('ConnectFour') {
                sh "trivy image rameshkumarverma/connectfour:latest > trivyimage.txt"
            }
            }
        }
        stage("deploy_docker"){
            steps{
              dir('ConnectFour') {
                sh "docker stop connectfour || true"  // Stop the container if it's running, ignore errors
                sh "docker rm connectfour || true" 
                sh "docker run -d --name connectfour -p 8080:80 rameshkumarverma/connectfour"
            }
            }
        }
      stage('Deploy to Kubernetes') {
            steps {
                script {
                    dir('ConnectFour') {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                            // Apply deployment and service YAML files
                            sh 'kubectl apply -f deployment.yml'
                            sh 'kubectl apply -f service.yml'

                            // Get the external IP or hostname of the service
                            // def externalIP = sh(script: 'kubectl get svc amazon-service -o jsonpath="{.status.loadBalancer.ingress[0].hostname}"', returnStdout: true).trim()

                            // Print the URL in the Jenkins build log
                            // echo "Service URL: http://${externalIP}/"
                        }
                    }
                }
            }
        }

    }
}
