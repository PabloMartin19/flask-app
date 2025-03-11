pipeline {
    environment {
        IMAGEN = "pablomartin19/flask-app" 
        LOGIN = 'USER_DOCKERHUB'
    }
    agent none
    stages {
        stage("Desarrollo") {
            agent {
                docker { 
                    image "python:3" 
                    args '-u root:root'
                }
            }
            stages {
                stage('Clone') {
                    steps {
                        git branch:'master', url:'https://github.com/PabloMartin19/flask-app.git'
                    }
                }
                stage('Install') {
                    steps {
                        sh 'pip install -r app/requirements.txt'
                    }
                }
                stage('Test') {
                    steps {
                        sh 'pytest app/test_app.py'
                    }
                }
            }
        }
        stage("Construccion") {
            agent any
            stages {
                stage('CloneAnfitrion') {
                    steps {
                        git branch:'master', url:'https://github.com/PabloMartin19/flask-app.git'
                    }
                }
                stage('BuildImage') {
                    steps {
                        script {
                            newApp = docker.build "$IMAGEN:latest"
                        }
                    }
                }
                stage('UploadImage') {
                    steps {
                        script {
                            docker.withRegistry('', LOGIN) {
                                newApp.push()
                            }
                        }
                    }
                }
                stage('RemoveImage') {
                    steps {
                        sh "docker rmi $IMAGEN:latest"
                    }
                }
                stage ('Deploy') {
                    steps {
                        sshagent(credentials: ['SSH_USER']) {
                            sh '''
                            ssh -o StrictHostKeyChecking=no pablo@dh.pablomartin.site <<EOF
                            if [ ! -d "flask-app" ]; then
                                git clone https://github.com/PabloMartin19/flask-app.git
                            fi
                            cd flask-app
                            git pull
                            export NOMBRE="Pablo"
                            docker-compose up -d --build
                            EOF
                            '''
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            mail to: 'pmartinhidalgo19@gmail.com',
            subject: "Status of pipeline: ${currentBuild.fullDisplayName}",
            body: "${env.BUILD_URL} has result ${currentBuild.result}" 
        }
    }
}
