

//https://jenkins.io/doc/book/pipeline/shared-libraries/#dynamic-retrieval
library identifier: 'toolslib@master', retriever: modernSCM(
  [$class: 'GitSCMSource',
   remote: 'https://github.com/lelandsindt/jenkins-pipeline-shared.git'])

pipeline {
  agent any
  options {
    disableConcurrentBuilds()
  }
  environment {
    //Use "Pipeline Utility Steps" plugin to read information from pom.xml into env variables
    //http://maven.apache.org/components/ref/3.3.9/maven-model/apidocs/org/apache/maven/model/Model.html
    POM_PROJECT_VERSION = readMavenPom().getVersion()
    POM_PROJECT_NAME = readMavenPom().getName()
    isRelease = check.isRelease(readMavenPom().getVersion())
    isSnapshot = check.isSnapshot(readMavenPom().getVersion())
    isReleaseCandidate = check.isReleaseCandidate(branch.toString())
    skipPipeline = check.skipPipeline(env.WORKSPACE)
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
      // Deploy RELEASE when branch is master and version is RELEASE
      when {
        branch 'master'
        expression { check.isRelease(env.POM_PROJECT_VERSION) }
        not { expression { check.skipPipeline(env.WORKSPACE) } }
      }
      steps {
        sh '''
          ./mvnw clean install
        '''
      }
    }
    stage('Maven Deploy SNAPSHOT ') {
      // Deploy SNAPSHOT when branch is not master and version is SNAPSHOT
      when {
        not { branch 'master' }
        expression { check.isSnapshot(env.POM_PROJECT_VERSION) }
        not { expression { check.skipPipeline(env.WORKSPACE) } }
      }
      steps {
        sh '''
          ./mvnw clean install
        '''
      }
    }
    stage('Docker Build ') {
      when {
        anyOf { // Did Maven Deploy Run? --- possible better solution: https://github.com/jenkinsci/declarative-pipeline-when-conditions-plugin 
          allOf {
            branch 'master'
            expression { check.isRelease(env.POM_PROJECT_VERSION) }
          }
          allOf {
            not { branch 'master' }
            expression { check.isSnapshot(env.POM_PROJECT_VERSION) }
          }
        } // Did Maven Deploy Run?
        not { expression { check.skipPipeline(env.WORKSPACE) } }
      }
      steps {
        sh '''
          echo "Docker Build"
        '''
      }
    }
    stage('Docker Push ') {
      when {
        anyOf { // Did Maven Deploy Run?
          allOf {
            branch 'master'
            expression { check.isRelease(env.POM_PROJECT_VERSION) }
          }
          allOf {
            not { branch 'master' }
            expression { check.isSnapshot(env.POM_PROJECT_VERSION) }
          }
        } // Did Maven Deploy Run?
        not { expression { check.skipPipeline(env.WORKSPACE) } }
      }
      steps {
        sh '''
          echo "Docker Push"
        '''
      }
    }
    stage('Deploy to Development') {
      when {
        anyOf { // Did Maven Deploy Run?
          allOf {
            branch 'master'
            expression { check.isRelease(env.POM_PROJECT_VERSION) }
          }
          allOf {
            not { branch 'master' }
            expression { check.isSnapshot(env.POM_PROJECT_VERSION) }
          }
        } // Did Maven Deploy Run?
        not { branch 'master' }
        expression { check.isSnapshot(env.POM_PROJECT_VERSION) }
        not { expression { check.skipPipeline(env.WORKSPACE) } }
        not { expression { check.isReleaseCandidate(env.GIT_BRANCH) } }
      }
      steps {
        sh ''' 
          echo "Deploy to development" 
        '''
      }
    }
    stage('Deploy to Intigration') {
      when {
        anyOf { // Did Maven Deploy Run?
          allOf {
            branch 'master'
            expression { check.isRelease(env.POM_PROJECT_VERSION) }
          }
          allOf {
            not { branch 'master' }
            expression { check.isSnapshot(env.POM_PROJECT_VERSION) }
          }
        } // Did Maven Deploy Run?
        expression { check.isReleaseCandidate(env.GIT_BRANCH) }
        expression { check.isSnapshot(env.POM_PROJECT_VERSION) }
        not { expression { check.skipPipeline(env.WORKSPACE) } }
      }
      steps {
        sh ''' 
          echo "Deploy to Intigration"
        '''
      }
    }
    stage('Deploy to Production') {
      when {
        anyOf { // Did Maven Deploy Run?
          allOf {
            branch 'master'
            expression { check.isRelease(env.POM_PROJECT_VERSION) }
          }
          allOf {
            not { branch 'master' }
            expression { check.isSnapshot(env.POM_PROJECT_VERSION) }
          }
        } // Did Maven Deploy Run?
        branch 'master' 
        expression { check.isRelease(env.POM_PROJECT_VERSION) } 
        not { expression { check.skipPipeline(env.WORKSPACE) } }
      }
      steps {
        input {
          message "Deploy to Production?"
        }
        sh ''' 
          echo "Deploy to Production"
        '''
      }
    }
  }
}
