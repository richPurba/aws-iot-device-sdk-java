#!groovy
pipeline {
  agent { label 'docker'}

  environment {
    // GENERAL VARIABLES
    CONTAINER_REGISTRY = 'harbor.linux.itv.local'
    HARBOR_CREDENTIALS = credentials('svc-jenkins')
     SVC_JENKINS_USER = credentials("svc_jenkins_user")
    //-------------------------------------------------
    // NAMESPACE = PREFIX-ENVIRONMENT
    PREFIX = 'feature-team'
    //-------------------------------------------------
    SERVICE = 'agl2dsh' // Name of the service to be deployed
    IMAGE = "$CONTAINER_REGISTRY/$PREFIX/$SERVICE"
    GIT_COMMIT= "latest"
    BUILD_TAG='0.4.5' //TODO: needed to be changed when releasing

  }

  parameters {
      choice(name: 'pipeline', choices: 'all\nrelease\nbuild-images\nupdate-secrets\ndeploy-kubernetes', description: 'Pipeline to run')
      choice(name: 'application', choices: 'all\nagl-forwarder\nmysql', description: 'application to use to deploy')
      choice(name: 'environment', choices: 'all\ntest\nprod', description: 'choose if you want to deploy on test, acc, prod or on all')
      choice(name: 'datacenter', choices: 'asd\nrtd', description: 'which datacenter you want to update')
  }

  stages {

    stage("init") {
      steps {
        // Testing if the pipeline as a valid syntax
        validateDeclarativePipeline 'Jenkinsfile'
      }
    }

    stage('Releasing Jar to Nexus') {
            // when { branch 'release/*' }
            when { expression { params.pipeline == 'release' || params.pipeline == 'all' }}
            steps {
                echo('These lines are debugging tool for the next build')
                sh 'git checkout master'
                sh 'git config remote.origin.url git@gitlabcd.itv.local:feature-team/aglforwarder.git'
                sh 'git remote -v'
                sh 'git config -l'
                sh 'git pull -X theirs|| head -c 100000 pom.xml'
                sh 'git status'
                sh 'mvn release:clean'

                echo('Prepare release')
                sh "mvn release:prepare -Dusername=${SVC_JENKINS_USER_USR} -Dpassword=${SVC_JENKINS_USER_PSW}"

                echo('Release')
                sh "mvn release:perform -Dusername=${SVC_JENKINS_USER_USR} -Dpassword=${SVC_JENKINS_USER_PSW}"

                echo('Cleaning up')
                sh "mvn release:clean"

                echo('Deploy RELEASE')
                sh "mvn deploy -Dmaven.test.skip=true -Dusername=${SVC_JENKINS_USER_USR} -Dpassword=${SVC_JENKINS_USER_PSW}"

            }
        }


  options {
    gitLabConnection('gitlab-itv')
    gitlabBuilds(builds: ['init', 'Releasing Jar to Nexus,','Maven Deploy SNAPSHOT','build images','deploy-kubernetes'])
  }
}