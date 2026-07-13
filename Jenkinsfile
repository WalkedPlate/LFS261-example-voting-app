pipeline {

  agent none

  stages {

    stage('worker-build') {
      when {
        changeset "**/worker/**"
      }

      agent {
        docker {
          image 'maven:3.9.8-sapmachine-21'
          args '-v $HOME/.m2:/root/.m2'
        }
      }

      steps {
        echo 'Compiling worker application...'

        dir('worker') {
          sh 'mvn compile'
        }
      }
    }

    stage('worker-test') {
      when {
        changeset "**/worker/**"
      }

      agent {
        docker {
          image 'maven:3.9.8-sapmachine-21'
          args '-v $HOME/.m2:/root/.m2'
        }
      }

      steps {
        echo 'Running unit tests on worker application...'

        dir('worker') {
          sh 'mvn clean test'
        }
      }
    }

    stage('worker-package') {
      when {
        branch 'master'
        changeset "**/worker/**"
      }

      agent {
        docker {
          image 'maven:3.9.8-sapmachine-21'
          args '-v $HOME/.m2:/root/.m2'
        }
      }

      steps {
        echo 'Packaging worker application...'

        dir('worker') {
          sh 'mvn package -DskipTests'

          archiveArtifacts(
            artifacts: '**/target/*.jar',
            fingerprint: true
          )
        }
      }
    }

    stage('worker-docker-package') {
      when {
        changeset "**/worker/**"
        branch 'master'
      }

      agent any

      steps {
        echo 'Packaging worker application with Docker...'

        script {
          docker.withRegistry(
            'https://index.docker.io/v1/',
            'dockerlogin'
          ) {
            def workerImage = docker.build(
              "arkelrak/worker:v${env.BUILD_ID}",
              "./worker"
            )

            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
            workerImage.push("latest")
          }
        }
      }
    }

    stage('result-build') {
      when {
        changeset "**/result/**"
      }

      agent {
        docker {
          image 'node:22.4.0-slim'
          args '--user root'
        }
      }

      steps {
        echo 'Installing result application dependencies...'

        dir('result') {
          sh 'npm ci'
        }
      }
    }

    stage('result-test') {
      when {
        changeset "**/result/**"
      }

      agent {
        docker {
          image 'node:22.4.0-slim'
          args '--user root'
        }
      }

      steps {
        echo 'Running unit tests on result application...'

        dir('result') {
          sh 'npm ci'
          sh 'npm test'
        }
      }
    }

    stage('result-docker-package') {
      when {
        changeset "**/result/**"
        branch 'master'
      }

      agent any

      steps {
        echo 'Packaging result application with Docker...'

        script {
          docker.withRegistry(
            'https://index.docker.io/v1/',
            'dockerlogin'
          ) {
            def resultImage = docker.build(
              "arkelrak/result:v${env.BUILD_ID}",
              "./result"
            )

            resultImage.push()
            resultImage.push("${env.BRANCH_NAME}")
            resultImage.push("latest")
          }
        }
      }
    }

    stage('vote-build') {
      when {
        changeset "**/vote/**"
      }

      agent {
        docker {
          image 'python:3.11-slim'
          args '--user root'
        }
      }

      steps {
        echo 'Installing vote application dependencies...'

        dir('vote') {
          sh 'pip install --no-cache-dir -r requirements.txt'
        }
      }
    }

    stage('vote-test') {
      when {
        changeset "**/vote/**"
      }

      agent {
        docker {
          image 'python:3.11-slim'
          args '--user root'
        }
      }

      steps {
        echo 'Running unit tests on vote application...'

        dir('vote') {
          sh 'pip install --no-cache-dir -r requirements.txt'
          sh 'nosetests -v'
        }
      }
    }

    stage('vote-docker-package') {
      when {
        changeset "**/vote/**"
      }

      agent any

      steps {
        echo 'Packaging vote application with Docker...'

        script {
          docker.withRegistry(
            'https://index.docker.io/v1/',
            'dockerlogin'
          ) {
            def branchTag = env.BRANCH_NAME.replace('/', '-')

            def voteImage = docker.build(
              "arkelrak/vote:v${env.BUILD_ID}",
              "./vote"
            )

            voteImage.push()
            voteImage.push(branchTag)
            voteImage.push("latest")
          }
        }
      }
    }

    stage('deploy to dev') {
      when {
        branch 'master'
      }

      agent any

      steps {
        echo 'Deploy instavote application with Docker Compose...'
        sh 'docker compose up -d'
      }
    }
  }

  post {
    always {
      echo 'Building mono pipeline for Instavote is completed.'
    }
  }
}
