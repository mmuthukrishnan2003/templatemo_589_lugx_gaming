pipeline {

    /****************************************************************
     * AGENT
     * Run this pipeline on any available Jenkins agent
     ****************************************************************/
    agent any


    /****************************************************************
     * PIPELINE OPTIONS
     ****************************************************************/
    options {

        // Skip automatic checkout
        skipDefaultCheckout(true)

        // Add timestamps to console output
        timestamps()

        // Keep only last 10 builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }


    /****************************************************************
     * BUILD PARAMETERS
     ****************************************************************/
    parameters {

        choice(
            name: 'BRANCH',
            choices: ['main', 'dev', 'preprod'],
            description: 'Select Git Branch'
        )
    }


    /****************************************************************
     * ENVIRONMENT VARIABLES
     ****************************************************************/
    environment {

        // Application Name
        APP_NAME = "templatemo_589_lugx_gaming"

        // Docker Image Name
        IMAGE_NAME = "mk2526/templatemo_589_lugx_gaming"

        // Docker Image Tag
        IMAGE = "${IMAGE_NAME}:${BUILD_NUMBER}"

        // GitHub Repository
        GIT_URL = "https://github.com/mmuthukrishnan2003/templatemo_589_lugx_gaming.git"

        // Jenkins Credentials
        GIT_CREDENTIALS = "github-credentials"
        DOCKER_CREDENTIALS = "dockerhub-credentials"
        SSH_CREDENTIALS = "deployment-ssh"

        // Deployment Server
        SERVER_IP = "172.16.0.111"
        SERVER_USER = "demo"

        // Port Mapping
        HOST_PORT = "3111"
        CONTAINER_PORT = "80"
    }


    stages {

        /************************************************************
         * CLEAN WORKSPACE
         ************************************************************/
        stage('Clean Workspace') {

            steps {

                echo "Cleaning Workspace..."

                cleanWs()
            }
        }


        /************************************************************
         * CHECKOUT SOURCE CODE
         ************************************************************/
        stage('Checkout') {

            steps {

                echo "Checkout Branch : ${params.BRANCH}"

                git(
                    branch: params.BRANCH,
                    credentialsId: env.GIT_CREDENTIALS,
                    url: env.GIT_URL
                )
            }
        }


        /************************************************************
         * BUILD DOCKER IMAGE
         ************************************************************/
        stage('Docker Build') {

            steps {

                echo "Building Docker Image : ${IMAGE}"

                sh """
                    docker build -t ${IMAGE} .

                    docker images | grep ${IMAGE_NAME}
                """
            }
        }


        /************************************************************
         * PUSH IMAGE TO DOCKER HUB
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
         * DEPLOY APPLICATION
         ************************************************************/
        stage('Deploy') {

            steps {

                sshagent([env.SSH_CREDENTIALS]) {

                    sh """
ssh -o StrictHostKeyChecking=no ${SERVER_USER}@${SERVER_IP} <<EOF

echo "===================================="
echo "Connected Successfully"
echo "===================================="

echo "Pull Latest Docker Image"
docker pull ${IMAGE}

echo "Stop Old Container"
docker stop ${APP_NAME} || true

echo "Remove Old Container"
docker rm ${APP_NAME} || true

echo "Run New Container"

docker run -d \
    --name ${APP_NAME} \
    --restart unless-stopped \
    -p ${HOST_PORT}:${CONTAINER_PORT} \
    ${IMAGE}

echo "Running Containers"
docker ps

echo "Deployment Completed"

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

                echo "Checking Application..."

                sh """
                    sleep 10

                    curl -I http://${SERVER_IP}:${HOST_PORT}
                """
            }
        }
    }


    /****************************************************************
     * POST BUILD ACTIONS
     ****************************************************************/
    post {

        success {

            echo """

=================================================

DEPLOYMENT SUCCESSFUL

Application URL

http://${SERVER_IP}:${HOST_PORT}

=================================================

"""
        }


        failure {

            echo """

=================================================

DEPLOYMENT FAILED

Check Jenkins Console Output

=================================================

"""
        }


        always {

            cleanWs()
        }
    }
}
