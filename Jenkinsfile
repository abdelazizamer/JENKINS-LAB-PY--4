pipeline {
    agent any
 
    environment {
        APP_NAME = 'app-hello-nti' 
        REPO_URL = "https://github.com/abdelazizamer/JENKINS-LAB-PY--4.git"
    }

    stages {
        stage('Getting Repo files') {
            steps {
                git branch: "main", credentialsId: 'jenkinsToken', url: "${REPO_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t ${APP_NAME}:${BUILD_NUMBER} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {

                        sh """
                            echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin

                            docker tag ${APP_NAME}:${BUILD_NUMBER} ${DOCKER_USERNAME}/${APP_NAME}:${BUILD_NUMBER}

                            docker push ${DOCKER_USERNAME}/${APP_NAME}:${BUILD_NUMBER}
                        """

                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                            # وقف واحذف الكونتينر القديم لو موجود عشان ما يحصلش conflict في البورت أو الاسم
                            docker stop "${APP_NAME}-running" || true
                            docker rm "${APP_NAME}-running" || true

                            # تشغيل الكونتينر الجديد باستخدام اسم المستخدم الصحيح
                            docker run -d -p 5000:5000 --name "${APP_NAME}-running" ${DOCKER_USERNAME}/${APP_NAME}:${BUILD_NUMBER}
                            
                            docker ps
                        """
                    }
                }
            }
        }
    }
}