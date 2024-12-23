pipeline {
    agent any

    environment {
        DOCKER_IMAGE_CAST = "julianricom1/jenkins_exam-cast"
        DOCKER_IMAGE_MOVIE = "julianricom1/jenkins_exam-movie"
        K8S_NAMESPACE_DEV = "dev"
        K8S_NAMESPACE_QA = "qa"
        K8S_NAMESPACE_STAGING = "staging"
        K8S_NAMESPACE_PROD = "prod"
        KUBECONFIG = credentials("config")
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out code from GitHub repository...'
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/julianricom1/Jenkins_Exam.git',
                        credentialsId: 'github-credentials-id'
                    ]]
                ])
            }
        }

        stage('Build and Push Docker Images') {
            parallel {
                stage('Build and Push Cast Service') {
                    steps {
                        dir('Jenkins_devops_exams/cast-service') {
                            echo 'Building and pushing Cast Service Docker image...'
                            sh "docker build -t ${DOCKER_IMAGE_CAST}:${env.BUILD_NUMBER} ."
                            sh "docker tag ${DOCKER_IMAGE_CAST}:${env.BUILD_NUMBER} ${DOCKER_IMAGE_CAST}:latest"
                            withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh """
                                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                                docker push ${DOCKER_IMAGE_CAST}:${env.BUILD_NUMBER}
                                docker push ${DOCKER_IMAGE_CAST}:latest
                                """
                            }
                        }
                    }
                }

                stage('Build and Push Movie Service') {
                    steps {
                        dir('Jenkins_devops_exams/movie-service') {
                            echo 'Building and pushing Movie Service Docker image...'
                            sh "docker build -t ${DOCKER_IMAGE_MOVIE}:${env.BUILD_NUMBER} ."
                            sh "docker tag ${DOCKER_IMAGE_MOVIE}:${env.BUILD_NUMBER} ${DOCKER_IMAGE_MOVIE}:latest"
                            withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh """
                                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                                docker push ${DOCKER_IMAGE_MOVIE}:${env.BUILD_NUMBER}
                                docker push ${DOCKER_IMAGE_MOVIE}:latest
                                """
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                echo 'Deploying to Dev environment...'
                sh "helm upgrade --install cast-service ./Jenkins_devops_exams/charts -n ${K8S_NAMESPACE_DEV} --set image.repository=${DOCKER_IMAGE_CAST},image.tag=${env.BUILD_NUMBER},service.nodePort=30100"
                sh "helm upgrade --install movie-service ./Jenkins_devops_exams/charts -n ${K8S_NAMESPACE_DEV} --set image.repository=${DOCKER_IMAGE_MOVIE},image.tag=${env.BUILD_NUMBER},service.nodePort=30101"
            }
        }

        stage('Deploy to QA') {
            steps {
                echo 'Deploying to QA environment...'
                sh "helm upgrade --install cast-service ./Jenkins_devops_exams/charts -n ${K8S_NAMESPACE_QA} --set image.repository=${DOCKER_IMAGE_CAST},image.tag=${env.BUILD_NUMBER},service.nodePort=30200"
                sh "helm upgrade --install movie-service ./Jenkins_devops_exams/charts -n ${K8S_NAMESPACE_QA} --set image.repository=${DOCKER_IMAGE_MOVIE},image.tag=${env.BUILD_NUMBER},service.nodePort=30201"
            }
        }

        stage('Deploy to Staging') {
            steps {
                echo 'Deploying to Staging environment...'
                sh "helm upgrade --install cast-service ./Jenkins_devops_exams/charts -n ${K8S_NAMESPACE_STAGING} --set image.repository=${DOCKER_IMAGE_CAST},image.tag=${env.BUILD_NUMBER},service.nodePort=30300"
                sh "helm upgrade --install movie-service ./Jenkins_devops_exams/charts -n ${K8S_NAMESPACE_STAGING} --set image.repository=${DOCKER_IMAGE_MOVIE},image.tag=${env.BUILD_NUMBER},service.nodePort=30301"
            }
        }

        stage('Deploy to Prod') {
            when {
                branch 'master'
            }
            steps {
                input message: 'Approve deployment to Production?', ok: 'Deploy'
                echo 'Deploying to Production environment...'
                sh "helm upgrade --install cast-service ./Jenkins_devops_exams/charts -n ${K8S_NAMESPACE_PROD} --set image.repository=${DOCKER_IMAGE_CAST},image.tag=${env.BUILD_NUMBER},service.nodePort=30400"
                sh "helm upgrade --install movie-service ./Jenkins_devops_exams/charts -n ${K8S_NAMESPACE_PROD} --set image.repository=${DOCKER_IMAGE_MOVIE},image.tag=${env.BUILD_NUMBER},service.nodePort=30401"
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline execution failed!'
        }
    }
}
