pipeline {

    /***************************************************************
     * AGENT
     * Run the pipeline on any available Jenkins agent.
     ***************************************************************/
    agent any

    /***************************************************************
     * OPTIONS
     ***************************************************************/
    options {
        skipDefaultCheckout(true)
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    /***************************************************************
     * PARAMETERS
     ***************************************************************/
    parameters {

        choice(
            name: 'BRANCH',
            choices: ['main', 'dev', 'preprod'],
            description: 'Select Git Branch'
        )

        choice(
            name: 'DEPLOY_SERVER',
            choices: ['SERVER1', 'SERVER2'],
            description: 'Select Deployment Server'
        )
    }

    /***************************************************************
     * ENVIRONMENT VARIABLES
     ***************************************************************/
    environment {

        APP_NAME = "templatemo_589_lugx_gaming"

        IMAGE_NAME = "mk2526/templatemo_589_lugx_gaming"

        IMAGE = "${IMAGE_NAME}:${BUILD_NUMBER}"

        GIT_URL = "https://github.com/mmuthukrishnan2003/templatemo_589_lugx_gaming.git"

        DOCKER_CREDENTIALS = "dockerhub-credentials"

        SSH_CREDENTIALS = "deployment-ssh"
    }

    /***************************************************************
     * STAGES
     ***************************************************************/
    stages {

        /***********************************************************
         * Checkout Source Code
         ***********************************************************/
        stage('Checkout') {

            steps {

                echo "Cloning repository..."

                git(
                    branch: params.BRANCH,
                    url: env.GIT_URL
                )

            }
        }

        /***********************************************************
         * Build Docker Image
         ***********************************************************/
        stage('Docker Build') {

            steps {

                echo "Building Docker image..."

                sh """
                    docker build -t ${IMAGE} .
                    docker image inspect ${IMAGE}
                """

            }
        }

        /***********************************************************
         * Login to Docker Hub and Push Image
         ***********************************************************/
        stage('Docker Login & Push') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: env.DOCKER_CREDENTIALS,
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh """
                        set -e

                        echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin

                        docker push ${IMAGE}

                        docker logout
                    """

                }

            }
        }

        /***********************************************************
         * Select Deployment Server
         ***********************************************************/
        stage('Select Deployment Server') {

            steps {

                script {

                    if (params.DEPLOY_SERVER == "SERVER1") {
                        env.SERVER_IP = "172.16.0.111"
                    } else {
                        env.SERVER_IP = "172.16.0.112"
                    }

                    echo "Deploy Server : ${SERVER_IP}"
                }

            }
        }

        /***********************************************************
         * Deploy Docker Container
         ***********************************************************/
        stage('Deploy') {

            steps {

                sshagent(credentials: [env.SSH_CREDENTIALS]) {

                    sh """
                    ssh -o StrictHostKeyChecking=no demo@${SERVER_IP} '
                        set -e

                        docker pull ${IMAGE}

                        docker stop ${APP_NAME} || true

                        docker rm ${APP_NAME} || true

                        docker run -d \
                            --name ${APP_NAME} \
                            --restart unless-stopped \
                            -p 80:80 \
                            ${IMAGE}

                        docker image prune -f
                    '
                    """

                }

            }
        }

        /***********************************************************
         * Verify Deployment
         ***********************************************************/
        stage('Health Check') {

            steps {

                sh """
                    sleep 15
                    curl --fail http://${SERVER_IP}
                """

            }
        }
    }

    /***************************************************************
     * POST ACTIONS
     ***************************************************************/
    post {

        success {

            echo "======================================="
            echo "Deployment Successful"
            echo "Application : ${APP_NAME}"
            echo "Image       : ${IMAGE}"
            echo "Server      : ${SERVER_IP}"
            echo "======================================="

        }

        failure {

            echo "======================================="
            echo "Deployment Failed"
            echo "Check Jenkins Console Output."
            echo "======================================="

        }

        always {

            cleanWs()

        }
    }
}
