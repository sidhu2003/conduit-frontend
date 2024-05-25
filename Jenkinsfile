def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger'
]

pipeline{
    agent any

   environment {
        registryUrl = 'programmer175/conduit_angular'
        registryCredential = 'programmer175'
        dockerImage = ''
    }

    stages{
        stage('Initialization & Unit Test') {
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

stage('Update Deployment file') {
    steps {
        sshagent(['GITHUB_SSH']) {
            sh '''
                rm -rf conduit-manifests
                git config --global user.email "sidhurv8@gmail.com"
                git config --global user.name "sidhu2003"
                git clone git@github.com:sidhu2003/conduit-manifests.git
                cd conduit-manifests
                sed -i 's|programmer175/conduit_angular:.*|programmer175/conduit_angular:'"$BUILD_NUMBER"'|' frontend_deployment.yaml
                git add frontend_deployment.yaml
                git commit -m "updating frontend image version to $BUILD_NUMBER"
                git push origin main
            '''
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
            echo 'Slack Notification'
            slackSend channel: '#app',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME}"
        }
    }
}