pipeline {

    /***************************************************************
     * AGENT
     * Run pipeline on Jenkins node
     ***************************************************************/
    agent any


    /***************************************************************
     * OPTIONS
     ***************************************************************/
    options {

        // Disable default Jenkins checkout
        skipDefaultCheckout(true)

        // Add timestamps in logs
        timestamps()

        // Keep only last 10 builds
        buildDiscarder(
            logRotator(
                numToKeepStr: '10'
            )
        )
    }



    /***************************************************************
     * PARAMETERS
     ***************************************************************/
    parameters {

        choice(
            name: 'BRANCH',
            choices: [
                'main',
                'dev',
                'preprod'
            ],
            description: 'Select Git Branch'
        )


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


        APP_NAME = "templatemo_589_lugx_gaming"


        IMAGE_NAME =
        "mk2526/templatemo_589_lugx_gaming"


        IMAGE =
        "${IMAGE_NAME}:${BUILD_NUMBER}"


        GIT_URL =
        "https://github.com/mmuthukrishnan2003/templatemo_589_lugx_gaming.git"


        GIT_CREDENTIALS =
        "github-credentials"


        DOCKER_CREDENTIALS =
        "dockerhub-credentials"


        SSH_CREDENTIALS =
        "deployment-ssh"


        HOST_PORT = "80"


        CONTAINER_PORT = "80"

    }



    stages {


        /***********************************************************
         * CHECKOUT CODE
         ***********************************************************/
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





        /***********************************************************
         * BUILD DOCKER IMAGE
         ***********************************************************/
        stage('Docker Build') {


            steps {


                sh """

                echo "Building Docker Image"

                docker build \
                -t ${IMAGE} .

                docker images

                """
            }
        }






        /***********************************************************
         * PUSH IMAGE TO DOCKER HUB
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
         * SELECT SERVER IP
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

                    Deployment Server:
                    ${SERVER_IP}

                    """

                }
            }
        }







        /***********************************************************
         * SSH CONNECTION TEST
         ***********************************************************/
        stage('SSH Test') {


            steps {


                sshagent([env.SSH_CREDENTIALS]) {


                    sh """

                    echo "Testing SSH"


                    ssh \
                    -o StrictHostKeyChecking=no \
                    demo@${SERVER_IP} "

                    whoami

                    hostname

                    docker ps

                    "

                    """
                }
            }
        }







        /***********************************************************
         * DEPLOY APPLICATION
         ***********************************************************/
        stage('Deploy') {


            steps {


                sshagent([env.SSH_CREDENTIALS]) {


                    sh """


                    ssh \
                    -o StrictHostKeyChecking=no \
                    demo@${SERVER_IP} << EOF



                    echo "Pull Docker Image"


                    docker pull ${IMAGE}




                    echo "Stop Existing Container"


                    docker stop ${APP_NAME} || true




                    echo "Remove Existing Container"


                    docker rm ${APP_NAME} || true





                    echo "Start New Container"



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
         * HEALTH CHECK
         ***********************************************************/
        stage('Health Check') {


            steps {


                sh """


                echo "Waiting Application Start"


                sleep 15



                curl --fail \
                http://${SERVER_IP}:80



                echo "Application Running Successfully"



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

            ==================================

            Deployment SUCCESS

            Application:
            ${APP_NAME}

            Image:
            ${IMAGE}

            Branch:
            ${BRANCH}

            Server:
            ${SERVER_IP}


            ==================================

            """
        }




        failure {


            echo """

            ==================================

            Deployment FAILED

            Check Jenkins Console

            ==================================

            """
        }




        always {


            cleanWs()

        }

    }

}
