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
              dir('Classifier-Using-JS') {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=classifier-using-js \
                    -Dsonar.projectKey=classifier-using-js'''
                }
              }
            }
        }
        stage("quality gate"){
           steps {
                script {
                  dir('Classifier-Using-JS') {
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
              dir('Classifier-Using-JS') {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
            }
        }
         stage('TRIVY FS SCAN') {
            steps {
              dir('Classifier-Using-JS') {
                sh "trivy fs . > trivyfs.txt"
            }
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                  dir('Classifier-Using-JS') {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t rameshkumarverma/classifier-using-js ."
                       // sh "docker tag classifier-using-js rameshkumarverma/classifier-using-js:latest"
                       sh "docker push rameshkumarverma/classifier-using-js:latest"
                    }
                  }
                }
            }
        }
        stage("TRIVY"){
            steps{
              dir('Classifier-Using-JS') {
                sh "trivy image rameshkumarverma/classifier-using-js:latest > trivyimage.txt"
            }
            }
        }
        stage("deploy_docker"){
            steps{
              dir('Classifier-Using-JS') {
                sh "docker stop classifier-using-js || true"  // Stop the container if it's running, ignore errors
                sh "docker rm classifier-using-js || true" 
                sh "docker run -d --name classifier-using-js -p 8080:80 rameshkumarverma/classifier-using-js"
            }
            }
        }
      stage('Deploy to Kubernetes') {
            steps {
                script {
                    dir('Classifier-Using-JS') {
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
