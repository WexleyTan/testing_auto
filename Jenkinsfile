pipeline {
    agent {
        label 'b'
    }
    tools {
        nodejs 'nodejs'
    }
    environment {
        IMAGE = "neathtan/auto_nextcd"
        DOCKER_IMAGE = "${IMAGE}:${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = "dockertoken"
        GIT_MANIFEST_REPO = "https://github.com/WexleyTan/auto_nextjs_manifest.git"
        GIT_BRANCH = "master"
        MANIFEST_REPO = "auto_nextjs_manifest"
        MANIFEST_FILE_PATH = "deployment.yaml"
        GIT_CREDENTIALS_ID = 'git_new'
        
    }
    stages {
        stage("checkout") {
            steps {
                echo "Running on $NODE_NAME"
                echo "${BUILD_NUMBER}"
                sh 'docker image prune --all -f'
                sh 'pwd'
                sh 'ls'
            }
        }

        stage("Build and Push Docker Image") {
            steps {
                script {
                    echo "Building Docker image..."
                    sh "docker build -t ${env.DOCKER_IMAGE} ."
                    sh "docker images | grep -i ${env.IMAGE}"
                    
                    echo "Pushing the image to Docker Hub"
                    withCredentials([usernamePassword(credentialsId: "${env.DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push ${env.DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage("Cloning the Manifest File") {
            steps {
                sh "pwd"
                sh "ls -l"
               
                echo "Checking if the manifest repository exists and removing it if necessary..."
                sh """
                    cd ..
                    pwd
                    echo "--------------------------"
                    echo "this is: ${env.GIT_MANIFEST_REPO}"
                    if [ -d "${env.MANIFEST_REPO}" ]; then
                      echo "DIRECTORY does exist."
                      rm -rf ${env.MANIFEST_REPO}
                    fi
                    git clone -b ${env.GIT_BRANCH} ${env.GIT_MANIFEST_REPO}
                    pwd
                    ls -l
                    echo "--------------------------------------------"
                 """
                echo "Cloning the manifest repository..."
            }
        }

        stage("Updating the Manifest File") {
            steps {
                script {
                    echo "Updating the image in the deployment manifest..."
                    sh """
                    cd ..
                    pwd
                    ls -l
                    echo "--------------------------------------------"
                    sed -i 's|image: ${env.IMAGE}:.*|image: ${env.DOCKER_IMAGE}|' ${env.MANIFEST_REPO}/${env.MANIFEST_FILE_PATH}
                    cat  ${env.MANIFEST_REPO}/${env.MANIFEST_FILE_PATH}
                    """
                }
            }
        }

        stage("Push Changes to the Manifest Repository") {
            steps {
                script {
                    dir("${env.MANIFEST_REPO}") {
                        withCredentials([usernamePassword(credentialsId: env.GIT_CREDENTIALS_ID, passwordVariable: 'GIT_PASS', usernameVariable: 'GIT_USER')]) {
                            sh """
                            cd ..
                            cd ${env.MANIFEST_REPO}
                            git config --global user.name "WexleyTan"
                            git config --global user.email "neathtan1402@gmail.com"
                            git branch
                            ls -l
                            pwd
                            echo "Start pushing to manifest repo"
                            git add .
                            git commit -m "Update image to ${env.DOCKER_IMAGE}"
                            git push https://${GIT_USER}:${GIT_PASS}@${GIT_MANIFEST_REPO}
                            """
                        }
                    }
                }
            }
        }
    }
}

