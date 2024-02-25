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
              dir('Covid 19 Awarness Website') {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=covid-19-awarness-website \
                    -Dsonar.projectKey=covid-19-awarness-website'''
                }
              }
            }
        }
        stage("quality gate"){
           steps {
                script {
                  dir('Covid 19 Awarness Website') {
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
              dir('Covid 19 Awarness Website') {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
            }
        }
         stage('TRIVY FS SCAN') {
            steps {
              dir('Covid 19 Awarness Website') {
                sh "trivy fs . > trivyfs.txt"
            }
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                  dir('Covid 19 Awarness Website') {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t rameshkumarverma/covid-19-awarness-website ."
                       // sh "docker tag covid-19-awarness-website rameshkumarverma/covid-19-awarness-website:latest"
                       sh "docker push rameshkumarverma/covid-19-awarness-website:latest"
                    }
                  }
                }
            }
        }
        stage("TRIVY"){
            steps{
              dir('Covid 19 Awarness Website') {
                sh "trivy image rameshkumarverma/covid-19-awarness-website:latest > trivyimage.txt"
            }
            }
        }
        stage("deploy_docker"){
            steps{
              dir('Covid 19 Awarness Website') {
                sh "docker stop covid-19-awarness-website || true"  // Stop the container if it's running, ignore errors
                sh "docker rm covid-19-awarness-website || true" 
                sh "docker run -d --name covid-19-awarness-website -p 8080:80 rameshkumarverma/covid-19-awarness-website"
            }
            }
        }
      stage('Deploy to Kubernetes') {
            steps {
                script {
                    dir('Covid 19 Awarness Website') {
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
