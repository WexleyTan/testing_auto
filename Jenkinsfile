pipeline {
    agent any
    tools {
        nodejs 'nodejs' 
    }
    environment {
        IMAGE = "neathtan/nextjs-adv"
        DOCKER_IMAGE = "${IMAGE}:${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = "dockertoken"
        GIT_MANIFEST_REPO = "https://github.com/WexleyTan/auto_nextjs_manifest.git"
        GIT_BRANCH = "master"
        MANIFEST_REPO = "auto_nextjs_manifest"
        MANIFEST_FILE_PATH = "deployment.yaml"
        GIT_CREDENTIALS_ID = 'git_pass'
    }

    stages {
        stage("Checkout") {
            steps {
                script {
                    echo "Running on $NODE_NAME"
                    echo "Build Number: ${BUILD_NUMBER}"
                    sh 'docker image prune --all -f' 
                }
            }
        }

        stage("Build and Push Docker Image") {
            steps {
                script {
                    echo "Building Docker image..."
                    sh "docker build -t ${env.DOCKER_IMAGE} ." 
                    
                    echo "Listing Docker images..."
                    sh "docker images | grep -i ${env.IMAGE}"
                    
                    echo "Pushing the image to Docker Hub..."
                    withCredentials([usernamePassword(credentialsId: "${env.DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin' 
                        sh "docker push ${env.DOCKER_IMAGE}" 
                    }
                }
            }
        }

        stage("Cloning the Manifest File") {
            steps {
                script {
                    echo "Checking if the manifest repository exists and removing it if necessary..."
                    sh """
                        if [ -d "${env.MANIFEST_REPO}" ]; then
                            echo "Directory ${env.MANIFEST_REPO} exists, removing it..."
                            rm -rf ${env.MANIFEST_REPO}
                        fi
                    """
                    
                    echo "Cloning the manifest repository..."
                    sh "git clone -b ${env.GIT_BRANCH} ${env.GIT_MANIFEST_REPO} ${env.MANIFEST_REPO}" 
                }
            }
        }

        stage("Updating the Manifest File") {
            steps {
                script {
                    echo "Updating the image in the deployment manifest..."
                    dir("${env.MANIFEST_REPO}") {
                        sh """
                            sed -i 's|image: ${env.IMAGE}:.*|image: ${env.DOCKER_IMAGE}|' ${env.MANIFEST_FILE_PATH} 
                            echo "Updated deployment file:"
                            cat ${env.MANIFEST_FILE_PATH} 
                        """
                        
                        echo "Committing and pushing changes to the manifest repository..."
                        withCredentials([usernamePassword(credentialsId: "${env.GIT_CREDENTIALS_ID}", passwordVariable: 'GIT_PASS', usernameVariable: 'GIT_USER')]) {
                            sh """
                                git config --global user.name "WexleyTan"
                                git config --global user.email "neathtan1402@gmail.com"
                                git add ${env.MANIFEST_FILE_PATH} 
                                git commit -m "Update image to ${DOCKER_IMAGE}" 
                                git push https://${GIT_USER}:${GIT_PASS}@github.com/WexleyTan/auto_nextjs_manifest.git ${GIT_BRANCH} 
                            """
                        }
                    }
                }
            }
        }
    }
}

