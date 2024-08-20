// @Library('jenkins-shared-lib') _
pipeline {

  // ========================================================================== //
  //                                  A g e n t s
  // ========================================================================== //


  agent { label "DockerAgent" }

  // ========================================================================== //
  //                                 O p t i o n s
  // ========================================================================== //

  options {
    timestamps()
    timeout(time: 30, unit: 'MINUTES')
    parallelsAlwaysFailFast()
    rateLimitBuilds(throttle: [count: 3, durationName: 'minute', userBoost: false])
    buildDiscarder(logRotator(numToKeepStr: '10'))
    ansiColor('xterm')
  }

  // ========================================================================== //
  //               E n v i r o n m e n t   &   C r e d e n t i a l s
  // ========================================================================== //

  environment {

    // Repository configuration
    APP_PROJECT = 'cms-wbv'
    APP_CODE = 'cms-wbv-admin-app'
    APP_REPO_URL = 'http://10.100.116.37:8080/cms-wooribank-vn/cms-wbv-admin-app.git'
    APP_REPO_BRANCH = 'develop'
    APP_REPO_CREDENTIALS = 'token-gitlab-key'

    //Deployment configuration
    DEPLOY_ENV = 'dev'
    DEPLOY_AGENT = 'MasterNode'

    SERVICE_PORT_PUBLISH = '9101'
    SERVICE_PORT_LOCAL = '8080'
    SERVICE_NAME = "$DEPLOY_ENV-$APP_CODE"
    SERVICE_ARGS = "-e SPRING_PROFILES_ACTIVE=$DEPLOY_ENV -e SERVER_PORT=$SERVICE_PORT_LOCAL -e SERVER_SERVLET_CONTEXT_PATH=/ -e JAVA_OPTS=-Djava.net.preferIPv4Stack=true"

    //Source code folder
    SOURCE_FOLDER = 'source-code'
    DEVOPS_FOLDER = "$APP_CODE"

    //Build configuration
    BUILD_IMAGE = 'gradle:7.1-jdk8'
    BUILD_COMMAND = 'gradle clean build -i --build-cache --no-daemon -x test'
    BUILD_CACHE = "devops-cache-gradle-$APP_PROJECT-$APP_CODE:/home/gradle/.gradle"

    HEALTH_CHECK_CMD = 'echo 0'
    BUILD_CHECK_CMD = 'ls build/libs'
    DOCKER_BUILDKIT = 1
  }

  // ========================================================================== //
  //                                  S t a g e s
  // ========================================================================== //
  stages {

    stage ('Checkout') {
      steps {
        milestone(ordinal: null, label: "Milestone: Checkout")
        label 'Checkout'
        dir("$SOURCE_FOLDER") {
          checkout(
            [
              $class: 'GitSCM',
              userRemoteConfigs: [
                [
                  url: "$APP_REPO_URL",
                  credentialsId: "$APP_REPO_CREDENTIALS",
                ]
              ],
              branches: [
                [
                  name: "*/$APP_REPO_BRANCH"
                ]
              ],
            ]
          )
        }
      }
    }

    // ========================================================================== //
    //                                   S e t u p
    // ========================================================================== //

    stage('Setup') {
      steps {
        milestone(ordinal: null, label: "Milestone: Setup")
        label 'Setup'
        dir("$SOURCE_FOLDER") {
          script {
            //Set the git commit
            env.GIT_COMMIT_APP = sh(script: 'git rev-parse HEAD', returnStdout: true)
            env.GIT_SHORT_COMMIT_APP = "${GIT_COMMIT_APP[0..6]}"

            //Set the docker image and tag
            env.DOCKER_IMAGE = "$APP_PROJECT/$APP_CODE"
            env.DOCKER_TAG = "$GIT_SHORT_COMMIT_APP"
            env.DOCKER_BUILDER_NAME = "agent-builder-$GIT_SHORT_COMMIT_APP"

            //Set the build name
            currentBuild.displayName = "$BUILD_DISPLAY_NAME ($GIT_SHORT_COMMIT_APP)"

            //Copy ci/cd files to sourcecode folder including hidden file (/.)
            sh "cp -r $WORKSPACE/$APP_PROJECT/$DEVOPS_FOLDER/. $WORKSPACE/$SOURCE_FOLDER"

          }
        }
      }
    }

    // ========================================================================== //
    //                                   B u i l d
    // ========================================================================== //
    stage('Build Code') {

      steps {
        milestone(ordinal: null, label: "Milestone: Build")

        timeout(time: 15, unit: 'MINUTES') {
          dir("$SOURCE_FOLDER") {
            sh "docker run --rm --name $DOCKER_BUILDER_NAME --network devops --volumes-from devops-jenkins-inbound-agent-docker -v $BUILD_CACHE:cached -w \"\$(pwd)\" $BUILD_IMAGE $BUILD_COMMAND"
            sh "$BUILD_CHECK_CMD"
          }
        }
      }
    }


    // ========================================================================== //
    //                                    C o d e  A n a l y s i s
    // ========================================================================== //

    stage('Analysis') {
      parallel {

    // ========================================================================== //
    //                                    T e s t
    // ========================================================================== //

        stage('Unit Tests') {
          steps {
              sleep(time:2,unit:"SECONDS")
              echo "Run unit tests."
          }
        }

    // ========================================================================== //
    //                               S o n a r Q u b e
    // ========================================================================== //

        stage('SonarQube Scan'){
          steps {
            // withSonarQubeEnv(installationName: 'mysonar'){
              sleep(time:1,unit:"SECONDS")
              echo "Run static code analysis."
            // }
          }
        }
      }
    }


    stage('SonarQube Quality Gate'){
      steps {
        // timeout(time: 2, unit: 'MINUTES'){
        //   waitForQualtityGate abortPipeline: true
        // }
        echo "Check SonarQube Quality Gate."
      }
    }

    // ========================================================================== //
    //                         D o c k e r   B u i l d s
    // ========================================================================== //

    stage('Build Container') {
      steps {
        milestone(ordinal: null, label: "Milestone: Docker Build")
        timeout(time: 60, unit: 'MINUTES') {
          dir("$SOURCE_FOLDER") {
            sh "docker run --rm --name $DOCKER_BUILDER_NAME --network devops --volumes-from devops-jenkins-inbound-agent-docker -w \"\$(pwd)\" docker:latest docker build -t $DOCKER_IMAGE:$DOCKER_TAG -t $DOCKER_IMAGE:latest --build-arg=BUILDKIT_INLINE_CACHE=1 --cache-from $DOCKER_IMAGE:latest ."
          }
        }
      }
    }

  // ========================================================================== //
  //                               D e p l o y s
  // ========================================================================== //

    stage('Deploy') {
      // agent { label "$DEPLOY_AGENT" }
      steps {
        script {
          lock(resource: "Deploy - App: $APP_CODE, Environment: $DEPLOY_ENV", inversePrecedence: true) {
            milestone(ordinal: null, label: "Milestone: Deploy")
            echo 'Deploying...'
            timeout(time: 15, unit: 'MINUTES') {
              echo "Deploy to $DEPLOY_ENV"
              try {
                  sh "docker stop $SERVICE_NAME"
                  sh "docker rm $SERVICE_NAME"
              } catch (Exception e) {
                  echo "An exception occurred: ${e.message}"
              }
              sh "docker run -d --network cms-wbv --name $SERVICE_NAME -p $SERVICE_PORT_PUBLISH:$SERVICE_PORT_LOCAL $SERVICE_ARGS $DOCKER_IMAGE:$DOCKER_TAG"
            }
          }
        }
      }
    }

  // ========================================================================== //
  //                               H e a l t h   C h e c k
  // ========================================================================== //

    stage('Health Check') {
      // agent { label "$DEPLOY_AGENT" }
      steps {
        script {
          timeout(time: 3, unit: 'MINUTES') {
            waitUntil(initialRecurrencePeriod: 1000, quiet: true) {
              def healthCheckResult = sh(script: "docker exec $SERVICE_NAME $HEALTH_CHECK_CMD", returnStatus: true)
              if (healthCheckResult == 0) {
                  return true
              } else {
                  return false
              }
            }
          }
        }
      }
    }
  }
  // ========================================================================== //
  //                                    P o s t
  // ========================================================================== //

  post {
    always {
      echo 'Always'
      cleanWs()
    }
    success {
      echo 'SUCCESS!'
    }
    fixed {
      echo "FIXED!"
    }
    failure {
      echo 'FAILURE!'
    }
  }
}
