pipeline {

    agent any

    /************************************************************
     * OPTIONS
     ************************************************************/
    options {
        skipDefaultCheckout(true)
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    /************************************************************
     * PARAMETERS
     ************************************************************/
    parameters {

        choice(
            name: 'BRANCH',
            choices: ['main', 'dev', 'preprod'],
            description: 'Select Git Branch'
        )

        choice(
            name: 'SERVER',
            choices: ['SERVER-1', 'SERVER-2'],
            description: 'Select Deployment Server'
        )
    }

    /************************************************************
     * ENVIRONMENT
     ************************************************************/
    environment {

        APP_NAME = "templatemo_589_lugx_gaming"

        IMAGE_NAME = "mk2526/templatemo_589_lugx_gaming"

        IMAGE = "${IMAGE_NAME}:${BUILD_NUMBER}"

        GIT_URL = "https://github.com/mmuthukrishnan2003/templatemo_589_lugx_gaming.git"

        GIT_CREDENTIALS = "github-credentials"

        DOCKER_CREDENTIALS = "dockerhub-credentials"

        SERVER_USER = "demo"

        HOST_PORT = "3111"

        CONTAINER_PORT = "80"
    }

    stages {

        /************************************************************
         * CLEAN
         ************************************************************/
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        /************************************************************
         * SELECT SERVER
         ************************************************************/
        stage('Select Deployment Server') {

            steps {

                script {

                    if (params.SERVER == "SERVER-1") {

                        env.SERVER_IP = "172.16.0.111"
                        env.SSH_CREDENTIAL = "deployment-server1"

                    } else {

                        env.SERVER_IP = "172.16.0.112"
                        env.SSH_CREDENTIAL = "deployment-server2"

                    }

                    echo """
==================================================

Selected Server : ${params.SERVER}

Server IP       : ${env.SERVER_IP}

SSH Credential  : ${env.SSH_CREDENTIAL}

==================================================
"""
                }
            }
        }

        /************************************************************
         * CHECKOUT
         ************************************************************/
        stage('Checkout') {

            steps {

                git(
                    branch: params.BRANCH,
                    credentialsId: env.GIT_CREDENTIALS,
                    url: env.GIT_URL
                )
            }
        }

        /************************************************************
         * BUILD IMAGE
         ************************************************************/
        stage('Docker Build') {

            steps {

                sh """
                    docker build -t ${IMAGE} .
                    docker images | grep ${IMAGE_NAME}
                """
            }
        }

        /************************************************************
         * PUSH IMAGE
         ************************************************************/
        stage('Docker Push') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: env.DOCKER_CREDENTIALS,
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh """

                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin

                        docker push ${IMAGE}

                        docker logout

                    """
                }
            }
        }

        /************************************************************
         * TEST SSH CONNECTION
         ************************************************************/
        stage('Test SSH') {

            steps {

                sshagent([env.SSH_CREDENTIAL]) {

                    sh """

                    ssh -o StrictHostKeyChecking=no ${SERVER_USER}@${SERVER_IP} "

                        echo 'Connected Successfully'

                        whoami

                        hostname

                        docker ps

                    "

                    """
                }
            }
        }

        /************************************************************
         * DEPLOY
         ************************************************************/
        stage('Deploy') {

            steps {

                sshagent([env.SSH_CREDENTIAL]) {

                    sh """

ssh -o StrictHostKeyChecking=no ${SERVER_USER}@${SERVER_IP} <<EOF

echo "Pull Image"

docker pull ${IMAGE}

echo "Stop Container"

docker stop ${APP_NAME} || true

echo "Remove Container"

docker rm ${APP_NAME} || true

echo "Start New Container"

docker run -d \\
--name ${APP_NAME} \\
--restart unless-stopped \\
-p ${HOST_PORT}:${CONTAINER_PORT} \\
${IMAGE}

echo "Running Containers"

docker ps

EOF

                    """
                }
            }
        }

        /************************************************************
         * HEALTH CHECK
         ************************************************************/
        stage('Health Check') {

            steps {

                sh """

                    sleep 10

                    curl -I http://${SERVER_IP}:${HOST_PORT}

                """
            }
        }
    }

    /************************************************************
     * POST
     ************************************************************/
    post {

        success {

            echo """

==================================================

DEPLOYMENT SUCCESSFUL

Application URL

http://${SERVER_IP}:${HOST_PORT}

==================================================

"""
        }

        failure {

            echo """

==================================================

DEPLOYMENT FAILED

==================================================

"""
        }

        always {

            cleanWs()
        }
    }
}
