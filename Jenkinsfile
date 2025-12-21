pipeline {
    agent any
    parameters {
        choice(name: 'ENV', choices: ["test", "dev", "prod"], description: 'Select the environment')
        string(name: 'FRONTEND_IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag')
        string(name: 'BACKEND_IMAGE_TAG', defaultValue: 'latest', description: 'Backend Docker image tag')
    }
    environment {
        // username/password credential with id 'GIT_PACKAGE'
        REGISTRY_CREDENTIALS = credentials('GIT_PACKAGE')
        GIT_REGISTRY = 'ghcr.io/kodecloud95'
        FRONTEND_IMAGE_NAME = "k8s-insight-frontend-${params.ENV}"
        BACKEND_IMAGE_NAME = "k8s-insight-backend-${params.ENV}"
    }
    stages {
        stage('Checkout Code') {
            steps {
                script {
                    if (params.ENV == "test") {
                        git branch: 'test', url: 'https://github.com/kodecloud95/K8s-Insight-Scoring-Matrix-Venkatesan.git'
                    } else if (params.ENV == "dev") {
                        git branch: 'dev', url: 'https://github.com/kodecloud95/K8s-Insight-Scoring-Matrix-Venkatesan.git'
                    } else if (params.ENV == "prod") {
                        git branch: 'prod', url: 'https://github.com/kodecloud95/K8s-Insight-Scoring-Matrix-Venkatesan.git'
                    }
                }
            }
        }

        stage('Build and Push Frontend Image') {
            steps {
                script {
                    // Use provided tag if non-empty, otherwise fall back to build number
                    def feTag = (params.FRONTEND_IMAGE_TAG ?: '').trim()
                    if (!feTag) { feTag = env.BUILD_NUMBER }
                    def frontendImage = "${GIT_REGISTRY}/${FRONTEND_IMAGE_NAME}:${feTag}"

                    sh """
                        docker build -f ./frontend/Dockerfile -t ${frontendImage} .
                        echo ${REGISTRY_CREDENTIALS_PSW} | docker login ghcr.io -u ${REGISTRY_CREDENTIALS_USR} --password-stdin
                        docker push ${frontendImage}
                    """
                }
            }
        }

        stage('Build and Push Backend Image') {
            steps {
                script {
                    // Use provided tag if non-empty, otherwise fall back to build number
                    def beTag = (params.BACKEND_IMAGE_TAG ?: '').trim()
                    if (!beTag) { beTag = env.BUILD_NUMBER }
                    def backendImage = "${GIT_REGISTRY}/${BACKEND_IMAGE_NAME}:${beTag}"

                    sh """
                        docker build -f ./backend/Dockerfile -t ${backendImage} .
                        echo ${REGISTRY_CREDENTIALS_PSW} | docker login ghcr.io -u ${REGISTRY_CREDENTIALS_USR} --password-stdin
                        docker push ${backendImage}
                    """
                }
            }
        }
    }
}