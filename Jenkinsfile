//https://jenkins.io/doc/book/pipeline/shared-libraries/#dynamic-retrieval
library identifier: 'toolslib@master', retriever: modernSCM(
  [$class: 'GitSCMSource',
   remote: 'https://github.com/lelandsindt/jenkins-pipeline-shared.git'])

pipeline {
  agent any
  environment {
    //Use "Pipeline Utility Steps" plugin to read information from pom.xml into env variables
    //http://maven.apache.org/components/ref/3.3.9/maven-model/apidocs/org/apache/maven/model/Model.html
    POM_PROJECT_VERSION = readMavenPom().getVersion()
    POM_PROJECT_NAME = readMavenPom().getName()
    isRelease = check.isRelease(readMavenPom().getVersion())
    isSnapshot = check.isSnapshot(readMavenPom().getVersion())
    isReleaseCandidate = check.isReleaseCandidate(branch.toString())
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
        expression { check.isRelease(readMavenPom().getVersion()) }
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
        expression { check.isSnapshot(readMavenPom().getVersion()) }
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
        not { branch 'master' }
        expression { check.isSnapshot(readMavenPom().getVersion()) }
        not { expression { check.skipPipeline(env.WORKSPACE) } }
      }
      steps {
        sh ''' 
          echo "Deploy to development" 
        '''
      }
    }
    stage('Deploy to Intigration') {
      when {
        expression { check.isReleaseCandidate(branch) }
        expression { check.isSnapshot(readMavenPom().getVersion()) }
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
        expression { check.isRelease(env.BRANCH) }
        not { expression { check.isSnapshot(readMavenPom().getVersion()) } }
        not { expression { check.skipPipeline(env.WORKSPACE) } }
      }
      steps {
        sh ''' 
          echo "Deploy to Production"
        '''
      }
    }
  }
}
