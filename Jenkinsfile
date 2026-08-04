pipeline {

    agent any

    /*****************************************************
     * TOOLS
     *****************************************************/

    /*****************************************************
     * PARAMETERS
     *****************************************************/
    parameters {

        choice(
            name: 'BRANCH',
            choices: ['dev', 'main', 'preprod'],
            description: 'Select Git Branch'
        )

        choice(
            name: 'DEPLOY_SERVER',
            choices: [
                'SERVER1',
                'SERVER2'
            ],
            description: 'Deployment Server'
        )

    }

    /*****************************************************
     * ENVIRONMENT
     *****************************************************/
    environment {

        APP_NAME = "bingo"

        IMAGE = "harbor.company.com/apps/bingo:${BUILD_NUMBER}"

        REGISTRY = "harbor.company.com"

        GIT_URL = "https://gitlab.company.com/project/bingo.git"

        GIT_CREDENTIALS = "gitlab-credentials"

        SONAR = "SonarQube"

        SSH_CREDENTIALS = "deployment-ssh"

        REGISTRY_CREDENTIALS = "harbor-credentials"

    }

    /*****************************************************
     * STAGES
     *****************************************************/

    stages {

        /**********************************************
         * Checkout
         **********************************************/
        stage('Checkout') {

            steps {

                git branch: params.BRANCH,
                    credentialsId: env.GIT_CREDENTIALS,
                    url: env.GIT_URL

            }
        }

        /**********************************************
         * Install
         **********************************************/
        stage('Install Dependencies') {

            steps {
                sh 'npm install'
            }

        }

        /**********************************************
         * Build
         **********************************************/
        stage('Build') {

            steps {

                sh 'npm run build'

            }

        }

        /**********************************************
         * Test
         **********************************************/
        stage('Test') {

            steps {

                sh 'npm test -- --watch=false'

            }

        }

        /**********************************************
         * SonarQube
         **********************************************/
        stage('SonarQube Analysis') {

            steps {

                withSonarQubeEnv("${SONAR}") {

                    sh """
                    sonar-scanner \
                    -Dsonar.projectKey=bingo \
                    -Dsonar.projectName=BINGO \
                    -Dsonar.sources=. \
                    """

                }

            }

        }

        /**********************************************
         * Quality Gate
         **********************************************/
        stage('Quality Gate') {

            steps {

                timeout(time: 10, unit: 'MINUTES') {

                    waitForQualityGate abortPipeline: true

                }

            }

        }

        /**********************************************
         * Trivy Scan
         **********************************************/
        stage('Security Scan') {

            steps {

                sh '''
                trivy fs . --exit-code 0
                '''

            }

        }

        /**********************************************
         * Docker Build
         **********************************************/
        stage('Docker Build') {

            steps {

                sh """
                docker build -t ${IMAGE} .
                """

            }

        }

        /**********************************************
         * Docker Push
         **********************************************/
        stage('Docker Push') {

            steps {

                withCredentials([usernamePassword(
                        credentialsId: REGISTRY_CREDENTIALS,
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                )]) {

                    sh """
                    echo \$PASS | docker login ${REGISTRY} -u \$USER --password-stdin

                    docker push ${IMAGE}

                    docker logout ${REGISTRY}
                    """

                }

            }

        }

        /**********************************************
         * Select Server
         **********************************************/
        stage('Select Deployment Server') {

            steps {

                script {

                    if(params.DEPLOY_SERVER == "SERVER1"){

                        env.SERVER_IP="172.16.0.111"

                    }else{

                        env.SERVER_IP="172.16.0.112"

                    }

                }

            }

        }

        /**********************************************
         * Deploy
         **********************************************/
        stage('Deploy') {

            steps {

                sshagent(credentials: [SSH_CREDENTIALS]) {

                    sh """

                    ssh -o StrictHostKeyChecking=no demo@${SERVER_IP} '

                    docker pull ${IMAGE}

                    docker stop ${APP_NAME} || true

                    docker rm ${APP_NAME} || true

                    docker run -d \
                    --name ${APP_NAME} \
                    --restart always \
                    -p 3000:3000 \
                    ${IMAGE}

                    '

                    """

                }

            }

        }

        /**********************************************
         * Health Check
         **********************************************/
        stage('Health Check') {

            steps {

                sh """

                sleep 20

                curl --fail http://${SERVER_IP}:3000/health

                """

            }

        }

    }

    /*****************************************************
     * POST
     *****************************************************/

    post {

        success {

            emailext(

                subject: "SUCCESS : ${JOB_NAME} #${BUILD_NUMBER}",

                body: """

Application : ${APP_NAME}

Branch : ${params.BRANCH}

Server : ${SERVER_IP}

Image : ${IMAGE}

Deployment Successful.

""",

                to: "admin@company.com"

            )

        }

        failure {

            sshagent(credentials: [SSH_CREDENTIALS]) {

                sh """

                ssh -o StrictHostKeyChecking=no demo@${SERVER_IP} '

                docker rollback ${APP_NAME} || true

                '

                """

            }

            emailext(

                subject: "FAILED : ${JOB_NAME} #${BUILD_NUMBER}",

                body: """

Deployment Failed.

Branch : ${params.BRANCH}

Server : ${SERVER_IP}

Rollback Attempted.

Please check Jenkins Console Logs.

""",

                to: "admin@company.com"

            )

        }

        always {

            cleanWs()

        }

    }

}
