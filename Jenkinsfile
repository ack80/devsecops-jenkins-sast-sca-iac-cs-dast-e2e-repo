pipeline {
    agent any
    tools {
        maven '3.8.7' // Ajusta el nombre según la configuración en Jenkins
    }

    stages {
        stage('Compile and Run Sonar Analysis') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    bat """mvn -Dmaven.test.failure.ignore verify sonar:sonar ^
                        -Dsonar.login=${env.SONAR_TOKEN} ^
                        -Dsonar.projectKey=easybuggy ^
                        -Dsonar.host.url=http://localhost:9000/"""
                }
            }
        }
        
        stage('Build') {
            steps {
                script {
                    docker.withRegistry('', 'dockerlogin') {
                        def app = docker.build("asecurityguru/testeb")
                    }
                }
            }
        }
        
        stage('Run Container Scan') {
            steps {
                withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                    script {
                        try {
                            bat("C:\\snyk\\snyk-win.exe container test asecurityguru/testeb")
                        } catch (err) {
                            echo err.getMessage()
                        }
                    }
                }
            }
        }
        
        stage('Run Snyk SCA') {
            steps {
                withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                    bat("mvn snyk:test -fn")
                }
            }
        }
        
        stage('Run DAST Using ZAP') {
            steps {
                bat("C:\\zap\\ZAP_2.12.0_Crossplatform\\ZAP_2.12.0\\zap.bat -port 9393 -cmd -quickurl https://www.example.com -quickprogress -quickout C:\\zap\\Output.html")
            }
        }

        stage('Checkov') {
            steps {
                bat("checkov -s -f main.tf")
            }
        }
    }
}
