pipeline {
    agent any

    tools {
        maven 'Maven 3.8.7' // Asegúrate que este nombre coincida con el definido en "Global Tool Configuration"
    } 

        stage('Compile and Run Sonar Analysis') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    script {
                    echo "Ejecutando análisis de SonarQube..."
                    def status = bat(
                    script: '''
                        echo TOKEN: %SONAR_TOKEN%
                        mvn -Dmaven.test.failure.ignore verify sonar:sonar ^
                            -Dsonar.login=%SONAR_TOKEN% ^
                            -Dsonar.projectKey=easybuggy ^
                            -Dsonar.host.url=http://localhost:9000/
                    ''',
                    returnStatus: true
                    )
                    if (status != 0) {
                        error "SonarQube analysis failed with exit code ${status}"
                    }
                }
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
                            bat '''
                                set SNYK_TOKEN=%SNYK_TOKEN%
                                C:\\snyk\\snyk-win.exe container test asecurityguru/testeb
                            '''
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
                    bat '''
                        set SNYK_TOKEN=%SNYK_TOKEN%
                        mvn snyk:test -fn
                    '''
                }
            }
        }
        
        stage('Run DAST Using ZAP') {
            steps {
                bat '''
                    C:\\zap\\ZAP_2.12.0_Crossplatform\\ZAP_2.12.0\\zap.bat -port 9393 -cmd -quickurl https://www.example.com -quickprogress -quickout C:\\zap\\Output.html
                '''
            }
        }

        stage('Checkov') {
            steps {
                bat "checkov -s -f main.tf"
            }
        }
    }
}
