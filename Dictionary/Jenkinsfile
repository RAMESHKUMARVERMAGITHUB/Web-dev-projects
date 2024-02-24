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
              dir('Dictionary') {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=dictionary \
                    -Dsonar.projectKey=dictionary'''
                }
              }
            }
        }
        stage("quality gate"){
           steps {
                script {
                  dir('Dictionary') {
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
              dir('Dictionary') {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
            }
        }
         stage('TRIVY FS SCAN') {
            steps {
              dir('Dictionary') {
                sh "trivy fs . > trivyfs.txt"
            }
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                  dir('Dictionary') {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t rameshkumarverma/dictionary ."
                       // sh "docker tag dictionary rameshkumarverma/dictionary:latest"
                       sh "docker push rameshkumarverma/dictionary:latest"
                    }
                  }
                }
            }
        }
        stage("TRIVY"){
            steps{
              dir('Dictionary') {
                sh "trivy image rameshkumarverma/dictionary:latest > trivyimage.txt"
            }
            }
        }
        stage("deploy_docker"){
            steps{
              dir('Dictionary') {
                sh "docker stop dictionary || true"  // Stop the container if it's running, ignore errors
                sh "docker rm dictionary || true" 
                sh "docker run -d --name dictionary -p 8080:80 rameshkumarverma/dictionary"
            }
            }
        }
      stage('Deploy to Kubernetes') {
            steps {
                script {
                    dir('Dictionary') {
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
