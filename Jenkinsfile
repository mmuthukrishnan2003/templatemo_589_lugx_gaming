pipeline {


    /***************************************************************
     * AGENT
     *
     * Run pipeline on Jenkins available node
     ***************************************************************/
    agent any



    /***************************************************************
     * PIPELINE OPTIONS
     ***************************************************************/
    options {


        // Disable automatic Jenkins SCM checkout
        skipDefaultCheckout(true)


        // Add timestamps in console output
        timestamps()


        // Keep only last 10 build history
        buildDiscarder(
            logRotator(
                numToKeepStr: '10'
            )
        )

    }




    /***************************************************************
     * USER PARAMETERS
     *
     * Values selected from Jenkins Build with Parameters
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

            description: 'Select Git Branch'

        )



        // Select deployment target server

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


        // Application/container name

        APP_NAME = "templatemo_589_lugx_gaming"



        // Docker Hub repository

        IMAGE_NAME = "mk2526/templatemo_589_lugx_gaming"



        // Docker image tag
        // Example:
        // mk2526/templatemo_589_lugx_gaming:15

        IMAGE = "${IMAGE_NAME}:${BUILD_NUMBER}"



        // Git repository URL

        GIT_URL =
        "https://github.com/mmuthukrishnan2003/templatemo_589_lugx_gaming.git"




        // Jenkins credential IDs

        GIT_CREDENTIALS =
        "github-credentials"



        DOCKER_CREDENTIALS =
        "dockerhub-credentials"



        SSH_CREDENTIALS =
        "deployment-ssh"




        // Application ports

        HOST_PORT = "80"

        CONTAINER_PORT = "80"


    }





    /***************************************************************
     * PIPELINE STAGES
     ***************************************************************/
    stages {



        /***********************************************************
         * STAGE 1
         *
         * Checkout source code from Git
         ***********************************************************/
        stage('Checkout') {


            steps {


                echo "Checking out branch ${params.BRANCH}"



                git(

                    branch: params.BRANCH,


                    // Remove this line if repository is public

                    credentialsId: env.GIT_CREDENTIALS,


                    url: env.GIT_URL

                )


            }

        }





        /***********************************************************
         * STAGE 2
         *
         * Build Docker Image
         *
         * Requires Dockerfile in repository
         ***********************************************************/
        stage('Docker Build') {


            steps {


                echo "Building Docker Image ${IMAGE}"



                sh """


                docker build \\
                -t ${IMAGE} .



                echo "Docker image created"



                docker images | grep ${IMAGE_NAME}


                """

            }

        }






        /***********************************************************
         * STAGE 3
         *
         * Push Docker Image to Docker Hub
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
                    docker login \\
                    -u \$DOCKER_USER \\
                    --password-stdin



                    docker push ${IMAGE}



                    docker logout



                    """

                }


            }

        }






        /***********************************************************
         * STAGE 4
         *
         * Select deployment server
         *
         * SERVER1 / SERVER2
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

                    Selected Server:
                    ${SERVER_IP}

                    """

                }


            }

        }







        /***********************************************************
         * STAGE 5
         *
         * Deploy Docker Container
         *
         * Actions:
         *
         * 1. SSH into server
         * 2. Pull latest image
         * 3. Stop old container
         * 4. Remove old container
         * 5. Start new container
         ***********************************************************/
        stage('Deploy') {



            steps {



                sshagent([env.SSH_CREDENTIALS]) {



                    sh """


                    ssh -o StrictHostKeyChecking=no \\
                    demo@${SERVER_IP} << EOF



                    echo "Connected to Deployment Server"



                    echo "Pulling Docker Image"



                    docker pull ${IMAGE}





                    echo "Stopping Existing Container"



                    docker stop ${APP_NAME} || true



                    echo "Removing Existing Container"



                    docker rm ${APP_NAME} || true






                    echo "Starting New Container"



                    docker run -d \\

                    --name ${APP_NAME} \\

                    -p ${HOST_PORT}:${CONTAINER_PORT} \\

                    --restart always \\

                    ${IMAGE}






                    echo "Cleaning Old Docker Images"



                    docker image prune -f





                    echo "Current Running Containers"



                    docker ps




EOF



                    """


                }


            }


        }








        /***********************************************************
         * STAGE 6
         *
         * Application Health Check
         ***********************************************************/
        stage('Health Check') {


            steps {



                sh """


                echo "Waiting for application startup"



                sleep 15




                curl --fail \\
                http://${SERVER_IP}





                echo "Application is Running Successfully"



                """


            }

        }



    }







    /***************************************************************
     * POST BUILD ACTIONS
     ***************************************************************/
    post {



        success {



            echo """


            ======================================

            Deployment Successful


            Application:
            ${APP_NAME}


            Image:
            ${IMAGE}


            Branch:
            ${BRANCH}


            Server:
            ${SERVER_IP}


            ======================================


            """

        }





        failure {



            echo """


            ======================================

            Deployment Failed


            Check Jenkins Console Logs


            ======================================


            """

        }





        always {


            // Remove workspace files

            cleanWs()

        }



    }


}
