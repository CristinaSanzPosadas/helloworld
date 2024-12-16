pipeline {
    agent none

    environment {
        PYTHON_PATH = '.'
        PYTHON_LOCAL = 'C:\\Users\\crist\\AppData\\Local\\Programs\\Python\\Python313\\python.exe'
        WIREMOCK_LOCAL = 'C:\\Users\\crist\\Wiremock\\wiremock-standalone-3.10.0.jar'
        WIREMOCK_DIR = 'C:\\Users\\crist\\Wiremock'
    }

    stages {

        stage('Echo') {
            agent {label 'principal'}
            steps {
                bat """
                whoami
                hostname
                echo ${env.WORKSPACE}
                """
                echo 'Ejecutando pipeline'
            }
        }

        stage('Git') {
            agent {label 'agent'}
            steps {
                bat """
                whoami
                hostname
                echo ${env.WORKSPACE}
                """
                git 'https://github.com/CristinaSanzPosadas/helloworld.git'
            }
        }
        
        stage('Verify code') {
            agent {label 'agent'}
            steps {
                bat """
                whoami
                hostname
                echo ${env.WORKSPACE}
                """
                bat 'dir'
            }
        }
        
        stage('Verify Workspace') {
            agent {label 'principal'}
            steps {
                bat """
                whoami
                hostname
                echo ${env.WORKSPACE}
                """
                echo "${env.WORKSPACE}"
            }
        }
        
        stage('Build') {
            agent {label 'principal'}
            steps {
                bat """
                whoami
                hostname
                echo ${env.WORKSPACE}
                """
                echo 'Construyendo'
            }
        }
        
        stage('Setup Virtual Environment') {
            agent {label 'agent'}
            steps {
                bat """
                whoami
                hostname
                echo ${env.WORKSPACE}
                """
                bat """
                ${env.PYTHON_LOCAL} -m venv venv
                venv\\Scripts\\pip install pytest flask
                """
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit testing') {
                    agent {label 'agent'}
                    steps {
                        bat """
                        whoami
                        hostname
                        echo ${env.WORKSPACE}
                        """
                        echo 'Ejecutando pruebas unitarias'
                        bat """
                        venv\\Scripts\\python -m pytest --junitxml=result-unit.xml test\\unit
                        """
                    }
                }
                stage('Service testing') {
                    agent {label 'agent'}
                    steps {
                        bat """
                        whoami
                        hostname
                        echo ${env.WORKSPACE}
                        """
                        echo 'Ejecutando pruebas de servicio'
                        script {
                            bat """
                            set FLASK_APP=app.api:api_application
                            start /B venv\\Scripts\\python -m flask run --port=5000
                            for /L %%i in (1,1,10) do (
                                netstat -an | findstr :5000 && exit 0 || ping -n 2 127.0.0.1 > nul
                            )
                            """
                        }
                        script {
                            bat """
                            start /B java -jar ${env.WIREMOCK_LOCAL} --port 9090 --root-dir=${env.WIREMOCK_DIR}
                            for /L %%i in (1,1,10) do (
                                netstat -an | findstr :9090 && exit 0 || ping -n 2 127.0.0.1 > nul
                            )
                            """
                        }
                        bat """
                        echo "Ejecutando pruebas..."
                        venv\\Scripts\\python -m pytest --junitxml=result-tests.xml test\\rest
                        """
                    }
                }
            }
        }

        stage('Results JUnit') {
            agent {label 'agent'}
            steps {
                bat """
                whoami
                hostname
                echo ${env.WORKSPACE}
                """
                junit 'result-tests.xml'
            }
        }
    }

    post {
        always {
            script {
                node {
                bat 'taskkill /f /im python.exe || echo "No Flask process running"'
                bat 'taskkill /f /im java.exe || echo "No Wiremock process running"'
                }
            }
            script {
                echo "Limpiando workspace..."
                node {
                    cleanWs()
                }
            }
        }
    }


}
