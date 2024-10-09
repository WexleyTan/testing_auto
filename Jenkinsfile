pipeline {
  agent { label 'b' }
  tools {
    nodejs 'nodejs'
  }
  stages {
    stage("build") {
      steps {
        echo "Building the application"
        sh 'docker build -t nextjs_jenkins .'
      }
    }

    stage("deploy") {
      steps { 
        echo 'Deploying the application'
        sh 'docker start nextjs_jenkins || docker run --name nextjs_jenkins -d -p 3001:3000 nextjs_jenkins'
        sh 'docker ps'
      }
    }
  }
}
