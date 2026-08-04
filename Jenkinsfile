pipeline {

    /****************************************************************
     * AGENT
     * Run pipeline on any available Jenkins Agent
     ****************************************************************/
    agent any


    /****************************************************************
     * PIPELINE OPTIONS
     ****************************************************************/
    options {

        // Do not checkout automatically
        skipDefaultCheckout(true)

        // Show timestamps in console
        timestamps()

        // Keep only last 10 builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }


    /****************************************************************
     * BUILD PARAMETERS
     ****************************************************************/
    parameters {

        // Git Branch
        choice(
            name: 'BRANCH',
            choices: ['main', 'dev', 'preprod'],
            description: 'Select Git Branch'
        )

        // Deployment Server
        choice(
            name: 'SERVER',
            choices: ['SERVER-1', 'SERVER-2'],
            description: 'Select Deployment Server'
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

        // Docker Image with Build Number
        IMAGE = "${IMAGE_NAME}:${BUILD_NUMBER}"

        // GitHub Repository
        GIT_URL = "https://github.com/mmuthukrishnan2003/templatemo_589_lugx_gaming.git"

        // Jenkins Credentials
        GIT_CREDENTIALS = "github-credentials"
        DOCKER_CREDENTIALS = "dockerhub-credentials"
        SSH_CREDENTIALS = "deployment-ssh"

        // SSH User
        SERVER_USER = "demo"

        // Docker Port Mapping
        HOST_PORT = "3111"
        CONTAINER_PORT = "80"
    }


    /****************************************************************
     * PIPELINE STAGES
     ****************************************************************/
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
         * SELECT DEPLOYMENT SERVER
         ************************************************************/
        stage('Select Deployment Server') {

            steps {

                script {

                    if (params.SERVER == "SERVER-1") {

                        env.SERVER_IP = "172.16.0.111"

                    } else if (params.SERVER == "SERVER-2") {

                        env.SERVER_IP = "172.16.0.112"

                    }

                    echo """

========================================

Selected Server : ${params.SERVER}

Server IP       : ${env.SERVER_IP}

========================================

"""
                }
            }
        }


        /************************************************************
         * CHECKOUT SOURCE CODE
         ************************************************************/
        stage('Checkout') {

            steps {

                echo "Checking out branch : ${params.BRANCH}"

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

                echo "Building Docker Image..."

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

echo "Pull Latest Image"
docker pull ${IMAGE}

echo "Stop Old Container"
docker stop ${APP_NAME} || true

echo "Remove Old Container"
docker rm ${APP_NAME} || true

echo "Run New Container"

docker run -d \\
    --name ${APP_NAME} \\
    --restart unless-stopped \\
    -p ${HOST_PORT}:${CONTAINER_PORT} \\
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

                echo "Waiting for application..."

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

Check Jenkins Console Output

==================================================

"""
        }

        always {

            cleanWs()
        }
    }
}
