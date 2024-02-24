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
              dir('JAVASCRIPT GAME') {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=javascript-game \
                    -Dsonar.projectKey=javascript-game'''
                }
              }
            }
        }
        stage("quality gate"){
           steps {
                script {
                  dir('JAVASCRIPT GAME') {
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
              dir('JAVASCRIPT GAME') {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
            }
        }
         stage('TRIVY FS SCAN') {
            steps {
              dir('JAVASCRIPT GAME') {
                sh "trivy fs . > trivyfs.txt"
            }
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                  dir('JAVASCRIPT GAME') {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t rameshkumarverma/javascript-game ."
                       // sh "docker tag javascript-game rameshkumarverma/javascript-game:latest"
                       sh "docker push rameshkumarverma/javascript-game:latest"
                    }
                  }
                }
            }
        }
        stage("TRIVY"){
            steps{
              dir('JAVASCRIPT GAME') {
                sh "trivy image rameshkumarverma/javascript-game:latest > trivyimage.txt"
            }
            }
        }
        stage("deploy_docker"){
            steps{
              dir('JAVASCRIPT GAME') {
                sh "docker stop javascript-game || true"  // Stop the container if it's running, ignore errors
                sh "docker rm javascript-game || true" 
                sh "docker run -d --name javascript-game -p 8080:80 rameshkumarverma/javascript-game"
            }
            }
        }
      stage('Deploy to Kubernetes') {
            steps {
                script {
                    dir('JAVASCRIPT GAME') {
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
