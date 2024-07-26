pipeline {
    agent any

    environment {
        IMAGE_TAG = "V.0.0.${env.BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = credentials('5f8b634a-148a-4067-b996-07b4b3276fba')
        DOCKERHUB_USERNAME = "idrisniyi94"
        DEPLOYMENT_NAME = "juice-shop"
        STAGING_IMAGE_NAME = "${DOCKERHUB_USERNAME}/${DEPLOYMENT_NAME}-staging:${IMAGE_TAG}"
        QA_IMAGE_NAME = "${DOCKERHUB_USERNAME}/${DEPLOYMENT_NAME}-qa:${IMAGE_TAG}"
        SANDBOX_IMAGE_NAME = "${DOCKERHUB_USERNAME}/${DEPLOYMENT_NAME}-sandbox:${IMAGE_TAG}"
        PRODUCTION_IMAGE_NAME = "${DOCKERHUB_USERNAME}/${DEPLOYMENT_NAME}-production:${IMAGE_TAG}"
        BRANCH_NAME = "${GIT_BRANCH.split('/')[1]}"
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage("Checkout") {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/staging'], [name: '*/qa'], [name: '*/sandbox'], [name: '*/production']], userRemoteConfigs: [[url: 'https://github.com/stwins60/juice-shop-demo.git']]])
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv('sonar-server') {
                        sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=juice-shop -Dsonar.projectName=juice-shop"
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    withSonarQubeEnv('sonar-server') {
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                    }
                }
            }
        }
        stage("OWASP vulnerability scanner") {
            steps {
                script {
                    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit --nvdApiKey f9c669af-b15f-4487-9d6d-d930c8f1b7a4   --delay=3000 --threads=4', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }
        stage("Docker login") {
            steps {
                script {
                    sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                    echo "Login Successful"
                }
            }
        }
        stage("Docker build") {
            steps {
                script {
                    if (BRANCH_NAME == 'staging') {
                        sh "docker build -t $STAGING_IMAGE_NAME ."
                    } else if (BRANCH_NAME == 'qa') {
                        sh "docker build -t $QA_IMAGE_NAME ."
                    } else if (BRANCH_NAME == 'sandbox') {
                        sh "docker build -t $SANDBOX_IMAGE_NAME ."
                    } else if (BRANCH_NAME == 'production') {
                        sh "docker build -t $PRODUCTION_IMAGE_NAME ."
                    }
                    
                }
            }
        }
        stage("Trivy Image Scan") {
            steps {
                script {
                    if (BRANCH_NAME == 'staging') {
                        sh "trivy image $STAGING_IMAGE_NAME"
                    } else if (BRANCH_NAME == 'qa') {
                        sh "trivy image $QA_IMAGE_NAME"
                    } else if (BRANCH_NAME == 'sandbox') {
                        sh "trivy image $SANDBOX_IMAGE_NAME"
                    } else if (BRANCH_NAME == 'production') {
                        sh "trivy image $PRODUCTION_IMAGE_NAME"
                    }
                    
                }
            }
        }
        stage("Docker Push") {
            steps {
                script {
                    if (BRANCH_NAME == 'staging') {
                        sh "docker push $STAGING_IMAGE_NAME"
                    } else if (BRANCH_NAME == 'qa') {
                        sh "docker push $QA_IMAGE_NAME"
                    } else if (BRANCH_NAME == 'sandbox') {
                        sh "docker push $SANDBOX_IMAGE_NAME"
                    } else if (BRANCH_NAME == 'production') {
                        sh "docker push $PRODUCTION_IMAGE_NAME"
                    }
                    
                }
            }
        }
    }
} 