pipeline {


    /***************************************************************
     * AGENT
     * Jenkins executes pipeline on available agent
     ***************************************************************/
    agent any



    /***************************************************************
     * OPTIONS
     ***************************************************************/
    options {

        // Disable default Jenkins checkout
        skipDefaultCheckout(true)


        // Show timestamps in console logs
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


        // Select Git branch during build

        choice(
            name: 'BRANCH',
            choices: [
                'main',
                'dev',
                'preprod'
            ],
            description: 'Select Git Branch'
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


        // Application name

        APP_NAME = "templatemo_589_lugx_gaming"



        // Docker Hub image repository

        IMAGE_NAME =
        "mk2526/templatemo_589_lugx_gaming"



        // Dynamic image tag

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



        // Docker ports

        HOST_PORT =
        "8080"


        CONTAINER_PORT =
        "80"


    }






    /***************************************************************
     * STAGES
     ***************************************************************/
    stages {



        /***********************************************************
         * CLEAN WORKSPACE
         ***********************************************************/
        stage('Clean Workspace') {


            steps {


                cleanWs()


            }

        }





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


                echo "Building Image : ${IMAGE}"



                sh """


                docker build \\
                -t ${IMAGE} .



                docker images | grep ${IMAGE_NAME}



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


                    echo \$DOCKER_PASS | docker login \\
                    -u \$DOCKER_USER \\
                    --password-stdin



                    docker push ${IMAGE}



                    docker logout



                    """

                }

            }

        }








        /***********************************************************
         * SELECT SERVER
         ***********************************************************/
        stage('Select Deployment Server') {


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

                    Deployment Server

                    ${SERVER_IP}

                    """

                }

            }

        }







        /***********************************************************
         * DEPLOY CONTAINER
         *
         * Steps:
         *
         * SSH server
         * Pull new image
         * Stop old container
         * Remove old container
         * Start new container
         ***********************************************************/
        stage('Deploy') {


            steps {


                sshagent([env.SSH_CREDENTIALS]) {



                    sh """


                    ssh -o StrictHostKeyChecking=no \\
                    demo@${SERVER_IP} << EOF



                    set -e



                    echo "Connected to Server"




                    echo "Pulling Docker Image"



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






                    echo "Removing unused images"



                    docker image prune -f





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


                echo "Waiting Application Startup"



                sleep 15




                curl --fail \\
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

            ==================================

            DEPLOYMENT SUCCESSFUL


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

            DEPLOYMENT FAILED


            Check Jenkins Console Output


            ==================================

            """

        }





        always {


            cleanWs()


        }


    }


}
