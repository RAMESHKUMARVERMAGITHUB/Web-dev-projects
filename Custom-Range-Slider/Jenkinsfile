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
              dir('Custom-Range-Slider') {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=custom-range-slider \
                    -Dsonar.projectKey=custom-range-slider'''
                }
              }
            }
        }
        stage("quality gate"){
           steps {
                script {
                  dir('Custom-Range-Slider') {
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
              dir('Custom-Range-Slider') {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
            }
        }
         stage('TRIVY FS SCAN') {
            steps {
              dir('Custom-Range-Slider') {
                sh "trivy fs . > trivyfs.txt"
            }
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                  dir('Custom-Range-Slider') {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t rameshkumarverma/custom-range-slider ."
                       // sh "docker tag custom-range-slider rameshkumarverma/custom-range-slider:latest"
                       sh "docker push rameshkumarverma/custom-range-slider:latest"
                    }
                  }
                }
            }
        }
        stage("TRIVY"){
            steps{
              dir('Custom-Range-Slider') {
                sh "trivy image rameshkumarverma/custom-range-slider:latest > trivyimage.txt"
            }
            }
        }
        stage("deploy_docker"){
            steps{
              dir('Custom-Range-Slider') {
                sh "docker stop custom-range-slider || true"  // Stop the container if it's running, ignore errors
                sh "docker rm custom-range-slider || true" 
                sh "docker run -d --name custom-range-slider -p 8080:80 rameshkumarverma/custom-range-slider"
            }
            }
        }
      stage('Deploy to Kubernetes') {
            steps {
                script {
                    dir('Custom-Range-Slider') {
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
