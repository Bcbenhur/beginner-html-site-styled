pipeline {
  agent { label 'docker' }
  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/Bcbenhur/beginner-html-site-styled.git', branch: 'gh-pages'
      }
    }

    stage('Build Image') {
      steps {
        sh 'docker build -t mdn-beginner-site:latest .'
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          docker rm -f website || true
          docker run -d --restart=always -p 99:80 --name website mdn-beginner-site:latest
        '''
      }
    }
  }

  post {
    always {
      echo "Clean up workspace"
      cleanWs()
    }
  }
}

