
// Syntax reference
// https://jenkins.io/doc/book/pipeline/syntax/#input

//https://jenkins.io/doc/book/pipeline/shared-libraries/#dynamic-retrieval
library identifier: 'toolslib@master', retriever: modernSCM(
  [$class: 'GitSCMSource',
   remote: 'https://github.com/GrowMon/jenkins-pipeline-shared.git'])

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
    skipPipeline = check.skipPipeline(this)
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
        not { environment name: 'BUILD_ID', value: '1' }
        not { expression { check.skipPipeline(this) } }
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
        not { environment name: 'BUILD_ID', value: '1' }
        not { expression { check.skipPipeline(this) } }
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
          allOf {                                                   // https://issues.jenkins-ci.org/browse/JENKINS-41187
            branch 'master'
            expression { check.isRelease(env.POM_PROJECT_VERSION) }
          }
          allOf {
            not { branch 'master' }
            expression { check.isSnapshot(env.POM_PROJECT_VERSION) }
          }
        } // Did Maven Deploy Run?
        not { environment name: 'BUILD_ID', value: '1' }
        not { expression { check.skipPipeline(this) } }
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
        not { environment name: 'BUILD_ID', value: '1' }
        not { expression { check.skipPipeline(this) } }
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
        branch 'develop'
        expression { check.isSnapshot(env.POM_PROJECT_VERSION) }
        not { expression { check.isReleaseCandidate(env.GIT_BRANCH) } }
        not { environment name: 'BUILD_ID', value: '1' }
        not { expression { check.skipDeploy(this) } }
        not { expression { check.skipPipeline(this) } }
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
        not { environment name: 'BUILD_ID', value: '1' }
        not { expression { check.skipDeploy(this) } }
        not { expression { check.skipPipeline(this) } }
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
        not { environment name: 'BUILD_ID', value: '1' }
        not { expression { check.skipDeploy(this) } }
        not { expression { check.skipPipeline(this) } }
      }
      //Todo: figure out how to get input go/nogo /after/ the when statment.
      //https://jenkins.io/doc/book/pipeline/syntax/#input
      // "... using the input step. The stage will pause after any options have
      // been applied, and before entering the stage`s `agent or evaluating its
      // when condition."
      // Also keep this in mind
      // https://stackoverflow.com/questions/46276140/jenkins-declarative-pipeline-user-input-parameters
      steps {
        sh ''' 
          echo "Deploy to Production"
        '''
      }
    }
  }
}
