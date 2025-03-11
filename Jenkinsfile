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
        stage("Construcción") {
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
                stage ('Despliegue') {
                    agent any
                    stages {
                        stage ('Despliegue en el VPS'){
                            steps {
                                sshagent(credentials : ['SSH_USER']) {
                                    sh '''ssh -o StrictHostKeyChecking=no pablo@dh.pablomartin.site "
                                    cd flask-app &&
                                    git pull &&
                                    docker-compose down &&
                                    docker pull pablomartin19/flask-app:latest &&
                                    docker-compose up -d &&
                                    docker image prune -f
                                    "'''
                                }
                            }
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
