pipeline {

    /***************************************************************
     * AGENT
     * Run this pipeline on any available Jenkins agent.
     ***************************************************************/
    agent any

    /***************************************************************
     * OPTIONS
     ***************************************************************/
    options {

        // Prevent Jenkins from checking out the repository twice
        skipDefaultCheckout(true)

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

        // Application Name
        APP_NAME = "templatemo_589_lugx_gaming"

        // Docker Hub Repository
        IMAGE_NAME = "mk2526/templatemo_589_lugx_gaming"

        // Docker Image
        IMAGE = "${IMAGE_NAME}:${BUILD_NUMBER}"

        // Git Repository
        GIT_URL = "https://github.com/mmuthukrishnan2003/templatemo_589_lugx_gaming.git"

        // Jenkins Credential IDs
        DOCKER_CREDENTIALS = "dockerhub-credentials"
        SSH_CREDENTIALS    = "deployment-ssh"

    }

    /***************************************************************
     * PIPELINE STAGES
     ***************************************************************/
    stages {

        /*******************************************************
         * CHECKOUT
         *******************************************************/
        stage('Checkout') {

            steps {

                echo "Cloning Repository..."

                git(
                    branch: params.BRANCH,
                    url: env.GIT_URL
                )

            }

        }

        /*******************************************************
         * BUILD DOCKER IMAGE
         *******************************************************/
        stage('Docker Build') {

            steps {

                echo "Building Docker Image..."

                sh """
                    docker build -t ${IMAGE} .
                """

            }

        }

        /*******************************************************
         * LOGIN & PUSH IMAGE
         *******************************************************/
        stage('Docker Login & Push') {

            steps {

                echo "Logging into Docker Hub..."

                withCredentials([
                    usernamePassword(
                        credentialsId: env.DOCKER_CREDENTIALS,
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker push '"${IMAGE}"'

                        docker logout
                    '''

                }

            }

        }

        /*******************************************************
         * SELECT DEPLOYMENT SERVER
         *******************************************************/
        stage('Select Deployment Server') {

            steps {

                script {

                    if (params.DEPLOY_SERVER == "SERVER1") {

                        env.SERVER_IP = "172.16.0.111"

                    } else {

                        env.SERVER_IP = "172.16.0.112"

                    }

                    echo "Deploying to ${env.SERVER_IP}"

                }

            }

        }

        /*******************************************************
         * DEPLOY APPLICATION
         *******************************************************/
        stage('Deploy') {

            steps {

                sshagent(credentials: [env.SSH_CREDENTIALS]) {

                    sh """
                    ssh -o StrictHostKeyChecking=no demo@${SERVER_IP} <<EOF

                    docker pull ${IMAGE}

                    docker stop ${APP_NAME} || true

                    docker rm ${APP_NAME} || true

                    docker run -d \\
                        --name ${APP_NAME} \\
                        --restart unless-stopped \\
                        -p 80:80 \\
                        ${IMAGE}

                    EOF
                    """

                }

            }

        }

        /*******************************************************
         * HEALTH CHECK
         *******************************************************/
        stage('Health Check') {

            steps {

                echo "Checking Application..."

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

            echo "========================================"
            echo "Deployment Successful"
            echo "========================================"

        }

        failure {

            echo "========================================"
            echo "Deployment Failed"
            echo "========================================"

        }

        always {

            cleanWs()

        }

    }

}
