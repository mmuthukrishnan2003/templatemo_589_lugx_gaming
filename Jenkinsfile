pipeline {

    /*****************************************************
     * AGENT
     * Runs the pipeline on any available Jenkins agent.
     *****************************************************/
    agent any

    /*****************************************************
     * PARAMETERS
     * Displayed in "Build with Parameters".
     *****************************************************/
    parameters {

        // Select the Git branch to build
        choice(
            name: 'BRANCH',
            choices: ['main', 'dev', 'preprod'],
            description: 'Git Branch'
        )

        // Select the deployment target server
        choice(
            name: 'DEPLOY_SERVER',
            choices: ['SERVER1', 'SERVER2'],
            description: 'Deployment Server'
        )
    }

    /*****************************************************
     * ENVIRONMENT VARIABLES
     * Global variables used throughout the pipeline.
     *****************************************************/
    environment {

        // Docker container name
        APP_NAME = "templatemo_589_lugx_gaming"

        // Docker Hub repository
        IMAGE_NAME = "mk2526/templatemo_589_lugx_gaming"

        // Docker image with Jenkins build number
        IMAGE = "${IMAGE_NAME}:${BUILD_NUMBER}"

        // GitHub repository URL
        GIT_URL = "https://github.com/mmuthukrishnan2003/templatemo_589_lugx_gaming.git"

        // Docker Hub credentials ID in Jenkins
        DOCKER_CREDENTIALS = "dockerhub-credentials"

        // SSH credentials ID for deployment server
        SSH_CREDENTIALS = "deployment-ssh"
    }

    /*****************************************************
     * PIPELINE STAGES
     *****************************************************/
    stages {

        /*************************************************
         * STAGE: Checkout
         * Clone the selected branch from GitHub.
         *************************************************/
        stage('Checkout') {

            steps {

                git branch: params.BRANCH,
                    url: env.GIT_URL

            }
        }

        /*************************************************
         * STAGE: Docker Build
         * Build the Docker image from the Dockerfile.
         *************************************************/
        stage('Docker Build') {

            steps {

                sh """
                    docker build -t ${IMAGE} .
                """

            }
        }

        /*************************************************
         * STAGE: Docker Login & Push
         * Login to Docker Hub and push the image.
         *************************************************/
        stage('Docker Login & Push') {

            steps {

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

        /*************************************************
         * STAGE: Select Deployment Server
         * Choose the destination server based on the
         * selected parameter.
         *************************************************/
        stage('Select Deployment Server') {

            steps {

                script {

                    if (params.DEPLOY_SERVER == "SERVER1") {

                        env.SERVER_IP = "172.16.0.111"

                    } else {

                        env.SERVER_IP = "172.16.0.112"

                    }

                }

            }
        }

        /*************************************************
         * STAGE: Deploy
         * Connect to the target server using SSH,
         * pull the latest Docker image,
         * stop/remove the old container,
         * and start the new container.
         *************************************************/
        stage('Deploy') {

            steps {

                sshagent(credentials: [env.SSH_CREDENTIALS]) {

                    sh """
                    ssh -o StrictHostKeyChecking=no demo@${SERVER_IP} << EOF

                    docker pull ${IMAGE}

                    docker stop ${APP_NAME} || true

                    docker rm ${APP_NAME} || true

                    docker run -d \\
                        --name ${APP_NAME} \\
                        --restart always \\
                        -p 80:80 \\
                        ${IMAGE}

                    EOF
                    """

                }

            }
        }

        /*************************************************
         * STAGE: Health Check
         * Wait for the application to start and
         * verify that the website is reachable.
         *************************************************/
        stage('Health Check') {

            steps {

                sh """
                    sleep 10

                    curl --fail http://${SERVER_IP}
                """

            }
        }

    }

    /*****************************************************
     * POST ACTIONS
     * Actions executed after the pipeline finishes.
     *****************************************************/
    post {

        /***********************************************
         * Executed when the pipeline succeeds.
         ***********************************************/
        success {

            echo "Deployment Successful"

        }

        /***********************************************
         * Executed when the pipeline fails.
         ***********************************************/
        failure {

            echo "Deployment Failed"

        }

        /***********************************************
         * Always executed regardless of success/failure.
         * Cleans the Jenkins workspace.
         ***********************************************/
        always {

            cleanWs()

        }

    }

}
