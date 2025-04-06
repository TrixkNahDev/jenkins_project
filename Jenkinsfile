pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') // ID des credentials DockerHub dans Jenkins
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'git@github.com:TrixkNahDev/jenkins_project.git', branch: 'main'
            }
        }
        stage('Build and Push Cast Service') {
            steps {
                dir('cast-service') {
                    sh 'docker build -t trixk/cast-service:${BUILD_NUMBER} .'
                    sh 'docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW'
                    sh 'docker push trixk/cast-service:${BUILD_NUMBER}'
                }
            }
        }
        stage('Build and Push Movie Service') {
            steps {
                dir('movie-service') {
                    sh 'docker build -t trixk/movie-service:${BUILD_NUMBER} .'
                    sh 'docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW'
                    sh 'docker push trixk/movie-service:${BUILD_NUMBER}'
                }
            }
        }
        stage('Deploy to Dev') {
            steps {
                sh 'helm upgrade --install cast-service ./charts --namespace dev --set image.repository=trixk/cast-service --set image.tag=${BUILD_NUMBER}'
                sh 'helm upgrade --install movie-service ./charts --namespace dev --set image.repository=trixk/movie-service --set image.tag=${BUILD_NUMBER}'
            }
        }
        stage('Deploy to QA') {
            when { branch 'qa' }
            steps {
                sh 'helm upgrade --install cast-service ./charts --namespace qa --set image.repository=trixk/cast-service --set image.tag=${BUILD_NUMBER}'
                sh 'helm upgrade --install movie-service ./charts --namespace qa --set image.repository=trixk/movie-service --set image.tag=${BUILD_NUMBER}'
            }
        }
        stage('Deploy to Staging') {
            when { branch 'staging' }
            steps {
                sh 'helm upgrade --install cast-service ./charts --namespace staging --set image.repository=trixk/cast-service --set image.tag=${BUILD_NUMBER}'
                sh 'helm upgrade --install movie-service ./charts --namespace staging --set image.repository=trixk/movie-service --set image.tag=${BUILD_NUMBER}'
            }
        }
        stage('Deploy to Prod') {
            when { branch 'master' }
            steps {
                input message: 'Deploy to Production?', ok: 'Yes'
                sh 'helm upgrade --install cast-service ./charts --namespace prod --set image.repository=trixk/cast-service --set image.tag=${BUILD_NUMBER}'
                sh 'helm upgrade --install movie-service ./charts --namespace prod --set image.repository=trixk/movie-service --set image.tag=${BUILD_NUMBER}'
            }
        }
    }
}