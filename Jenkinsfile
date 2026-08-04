pipeline {


    /***************************************************************
     * AGENT
     * Run pipeline on Jenkins worker
     ***************************************************************/
    agent any



    /***************************************************************
     * PIPELINE OPTIONS
     ***************************************************************/
    options {

        // Disable automatic checkout
        skipDefaultCheckout(true)


        // Add timestamps
        timestamps()


        // Keep only last 10 builds
        buildDiscarder(
            logRotator(
                numToKeepStr: '10'
            )
        )
    }



    /***************************************************************
     * BUILD PARAMETERS
     ***************************************************************/
    parameters {


        // Select Git branch
        choice(
            name: 'BRANCH',
            choices: [
                'main',
                'dev',
                'preprod'
            ],
            description: 'Select Branch'
        )



        // Select deployment server
        choice(
            name: 'DEPLOY_SERVER',
            choices: [
                'SERVER1',
                'SERVER2'
            ],
            description: 'Select Deployment Server'
        )

    }





    /***************************************************************
     * ENVIRONMENT VARIABLES
     ***************************************************************/
    environment {


        // Docker container name
        APP_NAME =
        "templatemo_589_lugx_gaming"



        // Docker Hub image
        IMAGE_NAME =
        "mk2526/templatemo_589_lugx_gaming"



        // Image tag based on Jenkins build number
        IMAGE =
        "${IMAGE_NAME}:${BUILD_NUMBER}"



        // Git repository
        GIT_URL =
        "https://github.com/mmuthukrishnan2003/templatemo_589_lugx_gaming.git"



        // Jenkins credentials

        GIT_CREDENTIALS =
        "github-credentials"



        DOCKER_CREDENTIALS =
        "dockerhub-credentials"



        SSH_CREDENTIALS =
        "deployment-ssh"



        // Application ports

        HOST_PORT =
        "80"



        CONTAINER_PORT =
        "80"

    }





    /***************************************************************
     * STAGES
     ***************************************************************/
    stages {



        /***********************************************************
         * 1. CHECKOUT SOURCE CODE
         ***********************************************************/
        stage('Checkout') {


            steps {


                echo "Checking branch ${params.BRANCH}"



                git(

                    branch: params.BRANCH,

                    credentialsId:
                    env.GIT_CREDENTIALS,

                    url:
                    env.GIT_URL

                )

            }

        }





        /***********************************************************
         * 2. BUILD DOCKER IMAGE
         ***********************************************************/
        stage('Docker Build') {


            steps {


                sh """

                echo "Building Docker Image"

                docker build \
                -t ${IMAGE} .


                echo "Docker Image Created"


                docker images

                """

            }

        }





        /***********************************************************
         * 3. PUSH IMAGE TO DOCKER HUB
         ***********************************************************/
        stage('Docker Push') {


            steps {


                withCredentials([


                    usernamePassword(

                        credentialsId:
                        env.DOCKER_CREDENTIALS,


                        usernameVariable:
                        'DOCKER_USER',


                        passwordVariable:
                        'DOCKER_PASS'

                    )

                ]) {



                    sh """

                    echo \$DOCKER_PASS |

                    docker login \
                    -u \$DOCKER_USER \
                    --password-stdin



                    docker push ${IMAGE}



                    docker logout


                    """

                }

            }

        }





        /***********************************************************
         * 4. SELECT SERVER
         ***********************************************************/
        stage('Server Selection') {


            steps {


                script {



                    if(params.DEPLOY_SERVER == "SERVER1") {


                        env.SERVER_IP =
                        "172.16.0.111"


                    }

                    else {


                        env.SERVER_IP =
                        "172.16.0.112"


                    }



                    echo """

                    Selected Deployment Server:

                    ${SERVER_IP}

                    """

                }

            }

        }







        /***********************************************************
         * 5. SSH CONNECTION TEST
         *
         * Verify Jenkins can connect to server
         ***********************************************************/
        stage('SSH Test') {


            steps {



                sshagent([env.SSH_CREDENTIALS]) {


                    sh """

                    echo "Testing SSH Connection"



                    ssh \
                    -o StrictHostKeyChecking=no \
                    demo@${SERVER_IP} "


                    echo User:

                    whoami



                    echo Host:

                    hostname



                    echo Docker:

                    docker ps


                    "


                    """

                }

            }

        }







        /***********************************************************
         * 6. DEPLOY APPLICATION
         ***********************************************************/
        stage('Deploy') {


            steps {


                sshagent([env.SSH_CREDENTIALS]) {



                    sh """


                    ssh \
                    -o StrictHostKeyChecking=no \
                    demo@${SERVER_IP} << EOF



                    echo "Connected"



                    echo "Pulling Image"


                    docker pull ${IMAGE}





                    echo "Stopping Old Container"



                    docker stop ${APP_NAME} || true





                    echo "Removing Old Container"



                    docker rm ${APP_NAME} || true






                    echo "Starting New Container"



                    docker run -d \\

                    --name ${APP_NAME} \\

                    -p ${HOST_PORT}:${CONTAINER_PORT} \\

                    --restart always \\

                    ${IMAGE}





                    echo "Deployment Completed"



                    docker ps



EOF



                    """

                }

            }

        }








        /***********************************************************
         * 7. APPLICATION HEALTH CHECK
         ***********************************************************/
        stage('Health Check') {


            steps {


                sh """


                echo "Waiting Application"



                sleep 15



                curl --fail \
                http://${SERVER_IP}:${HOST_PORT}



                echo "Application Healthy"



                """

            }

        }



    }





    /***************************************************************
     * POST ACTIONS
     ***************************************************************/
    post {


        success {


            echo """

            ===============================

            Deployment SUCCESS


            Application:
            ${APP_NAME}


            Image:
            ${IMAGE}


            Branch:
            ${BRANCH}


            Server:
            ${SERVER_IP}


            ===============================

            """

        }





        failure {


            echo """

            ===============================

            Deployment FAILED


            Check Jenkins Logs


            ===============================

            """

        }





        always {


            // Clean Jenkins workspace

            cleanWs()

        }

    }

}
