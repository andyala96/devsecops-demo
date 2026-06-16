pipeline {
    agent any

    stages {

        stage('GitLeaks Scan') {
            steps {
                sh 'gitleaks detect --source . --verbose'
            }
        }
        stage('OWASP Dependency Check') {
    steps {
        dependencyCheck additionalArguments: '--scan .',
                        odcInstallation: 'DependencyCheck'
    }
}

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('SonarQube') {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=devsecops-demo \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://localhost:9000
                        """
                    }
                }
            }
        }

stage('Quality Gate') {
    steps {
        echo 'Skipping Quality Gate'
    }
}

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t demoapp .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image --severity HIGH,CRITICAL demoapp'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker rm -f demoapp || true
                docker run -d --name demoapp -p 5000:5000 demoapp
                '''
            }
        }
    }
}
