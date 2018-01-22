//https://jenkins.io/doc/book/pipeline/shared-libraries/#dynamic-retrieval
library identifier: 'testing@master', retriever: modernSCM(
  [$class: 'GitSCMSource',
   remote: 'https://github.com/lelandsindt/jenkins-pipeline-shared.git'])

pipeline {
  agent any
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
        script {
          log.info 'nothing'
        }    
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
        sh '''
          echo "Docker Prune"
        '''
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
  environment {
    POM_PROJECT_VERSION = readMavenPom().getVersion()
    POM_PROJECT_NAME = readMavenPom().getName()
  }
}
