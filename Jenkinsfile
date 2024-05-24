pipeline{
    agent any

    tools {
        nodejs 'node'
    }

    environment {
        registryUrl = 'programmer175/conduit_angular'
        registryCredential = 'programmer175'
        dockerImage = ''
    }

    stages{
        stage('Initialization') {
            steps{
                sh 'npm install'
            }
        }

      stage('OWASP Dependency-Check Vulnerabilities') {
    environment {
        NVD_API_KEY = credentials('NVD_API_KEY')
    }
    steps {
        dependencyCheck additionalArguments: ''' 
                    -o './'
                    -s './'
                    -f 'ALL' 
                    --prettyPrint
                    --disableYarnAudit
                    --nvdApiKey ${NVD_API_KEY}
                    ''', odcInstallation: 'OWASP Dependency-Check Vulnerabilities'
        
        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
      }
}


        stage ('Build Docker Image') {
            steps {
                script {
                   dockerImage = docker.build registryUrl + ":$BUILD_NUMBER"
                }
            }
            
        }

        stage('Image Vulnerability Scan') {
            steps {
                script {
                    sh """trivy image --format template --template "@/usr/bin/html.tpl" --output trivy_report.html ${registryUrl}:${BUILD_NUMBER}""" 
                }
            }
        }

        stage ('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }

    }
     post {
        always {
            archiveArtifacts artifacts: "trivy_report.html", fingerprint: true
            publishHTML (target: [
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: '.',
                reportFiles: 'trivy_report.html',
                reportName: 'Trivy Scan',
                ])
            }
        }
}