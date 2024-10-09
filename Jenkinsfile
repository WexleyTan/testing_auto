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
        DOCKER_CREDENTIALS_ID = 'neathtan'
        GIT_MANIFEST_REPO = "https://github.com/WexleyTan/auto_nextjs_manifest"
        GIT_BRANCH = "master"
        MANIFEST_REPO = "manifest-repo"
        MANIFEST_FILE_PATH = "deployment.yaml"
        GIT_CREDENTIALS_ID = 'GIT_HUB'
        
    }
    stages {
      stage("checkout") {
        steps {
          echo "Running on $NODE_NAME"
          echo "${BUILD_NUMBER}"
          sh ' docker image prune --all '
          sh ' pwd '
          sh 'ls'
        }
      }

        stage("build and push docker image") {

            steps {
                script {
                    echo "Building docker image..."
                    sh ' docker build -t ${DOCKER_IMAGE} .'
                    sh ' docker images | grep -i ${IMAGE} '
                
                    echo "Pushing the image to Docker hub"
                    sh 'docker push ${DOCKER_IMAGE}'
                    
                }
            }
        }

        stage("Cloning the manifest file") {
            steps {
                sh "pwd"
                sh "ls -l"
                echo "Checking if the manifest repository exists and removing it if necessary..."
                sh '''
                    if [ -d "${MANIFEST_REPO}" ]; then
                        echo "${MANIFEST_REPO} exists, removing it..."
                        rm -rf ${MANIFEST_REPO}
                    fi
                '''
                echo "Updating the image of the Manifest file..."
                sh "git clone -b ${GIT_BRANCH} ${GIT_MANIFEST_REPO} ${MANIFEST_REPO}"
                sh "ls -l"
            }
        }

        stage("Updating the manifest file") {
            steps {
                script {
                    echo "Update the image in the deployment manifest..."
                    sh """
                    sed -i 's|image: ${IMAGE}:.*|image: ${DOCKER_IMAGE}|' ${MANIFEST_REPO}/${MANIFEST_FILE_PATH}
                    """
                }
            }
        }

        stage("push changes to the manifest") {
            steps {
                script {
                    dir("${MANIFEST_REPO}") {
                        withCredentials([usernamePassword(credentialsId: GIT_CREDENTIALS_ID, passwordVariable: 'GIT_PASS', usernameVariable: 'GIT_USER')]) {
                            sh """
                            git config --global user.name "WexleyTan"
                            git config --global user.email "neathtan1402@gmail.com"
                            git branch
                            ls -l 
                            pwd 

                            echo "Start pushing to manifest repo"
                            git add ${MANIFEST_FILE_PATH}
                            git commit -m "Update image to ${DOCKER_IMAGE}"
                            git push https://${GIT_USER}:${GIT_PASS}@github.com/WexleyTan/auto_nextjs_manifest
                            """
                        }
                    }
                }
            }
        }
        
  }
}
