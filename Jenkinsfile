pipeline {
  agent any
  environment {
    //Use "Pipeline Utility Steps" plugin to read information from pom.xml into env variables
    //http://maven.apache.org/components/ref/3.3.9/maven-model/apidocs/org/apache/maven/model/Model.html
    POM_PROJECT_VERSION = readMavenPom().getVersion()
    POM_PROJECT_NAME = readMavenPom().getName()
  }
  stages {
    stage('Initialize ') {
      steps {
        sh '''
          set +x
          echo " Building ${POM_PROJECT_NAME} "
          echo " ---------------------------------"
          set -x
          env
        '''
      }
    }
    stage('Compile') {
      when {
        anyOf {
          branch 'master'
          branch 'develop'
        }
      } 
      steps {
        sh '''
          ./mvnw clean install
        '''
      }
    }
    stage('Build Docker Containers') {
      when {
        anyOf {
          branch 'master'
          branch 'develop'
        }
      }
      steps {
        sh '''
          cd ./my-app/ 
          echo "docker build......" 
        '''
      }
    }
    stage('Docker Prune ') {
      steps {
        sh 'docker system prune --force '
      }
    }
    stage('Deploy to Development') {
      when {
        anyOf {
          branch 'develop'
          branch 'blueocean'
        }
        
      }
      steps {
        sh ''' 
          echo "Deploy to development" 
        '''
      }
    }
    stage('Deploy to Intigration') {
      when {
        anyOf {
          branch 'develop'
          branch 'blueocean'
        }
      }
      steps {
        sh ''' 
          echo "Deploy to Intigration"
        '''
      }
    }
    stage('Deploy to Production') {
      when {
        branch 'master'
      }
      steps {
        sh ''' 
          echo "Deploy to Production"
        '''
      }
    }
  }
}
