pipeline {
  agent any
  environment {
    //Use "Pipeline Utility Steps" plugin to read information from pom.xml into env variables
    //http://maven.apache.org/components/ref/3.3.9/maven-model/apidocs/org/apache/maven/model/Model.html
    POM_PROJECT_VERSION = readMavenPom().getVersion()
    POM_PROJECT_NAME = readMavenPom().getName()
  }
  parameters {
      choice(
          // choices are a string of newline separated values
          // https://issues.jenkins-ci.org/browse/JENKINS-41180
          choices: 'greeting\nsilence',
          description: '',
          name: 'REQUESTED_ACTION')
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
    stage('Maven Deploy RELEASE ') {
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
    stage('Maven Deploy SNAPSHOT ') {
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
