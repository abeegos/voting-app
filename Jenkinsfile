pipeline {
    agent none

    stages {
        stage('worker build') {
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset '**/worker/**'
            }
            steps {
                echo 'Compiling worker app'
                dir('worker') {
                    sh 'mvn compile'
                }
            }
        }

        stage('worker test') {
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset '**/worker/**'
            }
            steps {
                echo 'Running unit test on worker app'
                dir('worker') {
                    sh 'mvn clean test'
                }
            }
        }

        stage('worker-docker-package') {
            agent any
            when {
                branch 'master'
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging worker app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("systemswrangler/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }

        stage('vote build') {
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when {
                changeset '**/vote/**'
            }
            steps {
                echo 'Compile vote app'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                }
            }
        }

        stage('vote test') {
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when {
                changeset '**/vote/**'
            }
            steps {
                echo 'Running Unit Tests on vote app'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }

        stage('vote-docker-package') {
            agent any
            when {
                branch 'master'
                changeset "**/vote/**"
            }
            steps {
                echo 'Packaging vote app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("systemswrangler/vote:v${env.BUILD_ID}", "./vote")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }


        stage('result build') {
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when {
                changeset '**/result/**'
            }
            steps {
                echo 'Building result app'
                dir('result') {
                    sh 'npm install'
                }
            }
        }

        stage('result test') {
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when {
                changeset '**/result/**'
            }
            steps {
                echo 'Running Unit Tests on result app'
                dir('result') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

        stage('result-docker-package') {
            agent any
            when {
                branch 'master'
                changeset "**/result/**"
            }
            steps {
                echo 'Packaging result app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("systemswrangler/result:v${env.BUILD_ID}", "./result")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline for voting monorepo app is completed'
        }
        failure {
            slackSend(channel: "voting-cd", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success {
            slackSend(channel: "voting-cd", message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
    }
}