pipeline {
    agent any

    environment {
        PYTHON_PATH = '.'
        PYTHON_LOCAL = 'C:\\Users\\crist\\AppData\\Local\\Programs\\Python\\Python313\\python.exe'
        WIREMOCK_LOCAL = 'C:\\Users\\crist\\Wiremock\\wiremock-standalone-3.10.0.jar'
        WIREMOCK_DIR = 'C:\\Users\\crist\\Wiremock'
    }

    stages {

        stage('Echo') {
            steps {
                echo 'Ejecutando pipeline'
            }
        }

        stage('Git') {
            steps {
                git 'https://github.com/CristinaSanzPosadas/helloworld.git'
            }
        }
        
        stage('Verify code') {
            steps {
                bat 'dir'
            }
        }
        
        stage('Verify Workspace') {
            steps {
                echo "${env.WORKSPACE}"
            }
        }
        
        stage('Build') {
            steps {
                echo 'Construyendo'
            }
        }
        
        stage('Setup Virtual Environment') {
            steps {
                bat """
                ${env.PYTHON_LOCAL} -m venv venv
                venv\\Scripts\\pip install pytest flask
                """
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit testing') {
                    steps {
                        echo 'Ejecutando pruebas unitarias'
                        bat """
                        venv\\Scripts\\python -m pytest --junitxml=result-unit.xml test\\unit
                        """
                    }
                }
                stage('Service testing') {
                    steps {
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
                        echo "Ejecutando pruebas"
                        venv\\Scripts\\python -m pytest --junitxml=result-tests.xml test\\rest
                        """
                    }
                }
            }
        }

        stage('Results JUnit') {
            steps {
                junit 'result-tests.xml'
            }
        }
    }
}
