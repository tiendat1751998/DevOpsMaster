// // @Library('jenkins-shared-lib') _
// pipeline {
//
//   // ========================================================================== //
//   //                                  A g e n t s
//   // ========================================================================== //
//
//
//   agent any
//
//   // ========================================================================== //
//   //                                 O p t i o n s
//   // ========================================================================== //
// //
// //   options {
// //     timestamps()
// //     timeout(time: 30, unit: 'MINUTES')
// //     parallelsAlwaysFailFast()
// //     rateLimitBuilds(throttle: [count: 3, durationName: 'minute', userBoost: false])
// //     buildDiscarder(logRotator(numToKeepStr: '10'))
// //     ansiColor('xterm')
// //   }
//
//   // ========================================================================== //
//   //               E n v i r o n m e n t   &   C r e d e n t i a l s
//   // ========================================================================== //
//
//   environment {
//
//     // Repository configuration
//     APP_PROJECT = 'dotiendat1751998'
//     APP_CODE = 'devopsmater'
//     APP_REPO_URL = 'git@github.com:tiendat1751998/DevOpsMaster.git'
//     APP_REPO_BRANCH = 'master'
//     APP_REPO_CREDENTIALS = 'cms-git-key'
//
//     //Deployment configuration
//     DEPLOY_ENV = 'dev'
//     DEPLOY_AGENT = 'MasterNode'
//
//     SERVICE_PORT_PUBLISH = '8087'
//     SERVICE_PORT_LOCAL = '8080'
//     SERVICE_NAME = "$DEPLOY_ENV-$APP_CODE"
//     SERVICE_ARGS = "-e SPRING_PROFILES_ACTIVE=$DEPLOY_ENV -e SERVER_PORT=$SERVICE_PORT_LOCAL -e SERVER_SERVLET_CONTEXT_PATH=/ -e JAVA_OPTS=-Djava.net.preferIPv4Stack=true"
//     SERVICE_MEM_LIMIT = '1g'
//
//     //Source code folder
//     SOURCE_FOLDER = 'source-code'
//     DEVOPS_FOLDER = "$APP_CODE"
//
//     //Logs folder
//     LOGS_FOLDER = '/home/InfoCMS/logs/api_app'
//     LOGS_SOURCE = '/usr/local/InfoCMS/was/logs'
//
//     //Build configuration
//     //  need import image to docker
//     BUILD_IMAGE = 'gradle:7.1-jdk8'
//     BUILD_COMMAND = 'gradle clean build -i --build-cache --no-daemon -x test'
//     BUILD_CACHE = "devops-cache-gradle-$APP_PROJECT-$APP_CODE:/home/gradle/.gradle"
//
//     HEALTH_CHECK_CMD = 'echo 0'
//     BUILD_CHECK_CMD = 'ls build/libs'
//     DOCKER_BUILDKIT = 1
//   }
//
//   // ========================================================================== //
//   //                                  S t a g e s
//   // ========================================================================== //
//   stages {
//
//     stage ('Checkout') {
//       steps {
//         milestone(ordinal: null, label: "Milestone: Checkout")
//         label 'Checkout'
//         dir("$SOURCE_FOLDER") {
//           checkout(
//             [
//               $class: 'GitSCM',
//               userRemoteConfigs: [
//                 [
//                   url: "$APP_REPO_URL",
//                   credentialsId: "$APP_REPO_CREDENTIALS",
//                 ]
//               ],
//               branches: [
//                 [
//                   name: "*/$APP_REPO_BRANCH"
//                 ]
//               ],
//             ]
//           )
//         }
//       }
//     }
//
//     // ========================================================================== //
//     //                                   S e t u p
//     // ========================================================================== //
//
//     stage('Setup') {
//       steps {
//         milestone(ordinal: null, label: "Milestone: Setup")
//         label 'Setup'
//         dir("$SOURCE_FOLDER") {
//           script {
//             //Set the git commit
//             env.GIT_COMMIT_APP = sh(script: 'git rev-parse HEAD', returnStdout: true)
//             env.GIT_SHORT_COMMIT_APP = "${GIT_COMMIT_APP[0..6]}"
//
//             //Set the docker image and tag
//             env.DOCKER_IMAGE = "$APP_PROJECT/$APP_CODE"
//             env.DOCKER_TAG = "$GIT_SHORT_COMMIT_APP"
//             env.DOCKER_BUILDER_NAME = "agent-builder-$GIT_SHORT_COMMIT_APP"
//
//             //Set the build name
//             currentBuild.displayName = "$BUILD_DISPLAY_NAME ($GIT_SHORT_COMMIT_APP)"
//
//             //Copy ci/cd files to sourcecode folder including hidden file (/.)
//             sh "cp -r $WORKSPACE/$APP_PROJECT/$DEVOPS_FOLDER/. $WORKSPACE/$SOURCE_FOLDER"
//
//           }
//         }
//       }
//     }
//
//     // ========================================================================== //
//     //                                   B u i l d
//     // ========================================================================== //
//     stage('Build Code') {
//
//       steps {
//         milestone(ordinal: null, label: "Milestone: Build")
//
//         timeout(time: 15, unit: 'MINUTES') {
//           dir("$SOURCE_FOLDER") {
//             sh "docker run --rm --name $DOCKER_BUILDER_NAME --network devops --volumes-from devops-jenkins -v $BUILD_CACHE:cached -w \"\$(pwd)\" $BUILD_IMAGE $BUILD_COMMAND"
//             sh "$BUILD_CHECK_CMD"
//           }
//         }
//       }
//     }
//
//
//     // ========================================================================== //
//     //                                    C o d e  A n a l y s i s
//     // ========================================================================== //
//
//     stage('Analysis') {
//       parallel {
//
//     // ========================================================================== //
//     //                                    T e s t
//     // ========================================================================== //
//
//         stage('Unit Tests') {
//           steps {
//               sleep(time:2,unit:"SECONDS")
//               echo "Run unit tests."
//           }
//         }
//
//     // ========================================================================== //
//     //                               S o n a r Q u b e
//     // ========================================================================== //
//
//         stage('SonarQube Scan'){
//           steps {
//             // withSonarQubeEnv(installationName: 'mysonar'){
//               sleep(time:1,unit:"SECONDS")
//               echo "Run static code analysis."
//             // }
//           }
//         }
//       }
//     }
//
//
//     stage('SonarQube Quality Gate'){
//       steps {
//         // timeout(time: 2, unit: 'MINUTES'){
//         //   waitForQualtityGate abortPipeline: true
//         // }
//         echo "Check SonarQube Quality Gate."
//       }
//     }
//
//     // ========================================================================== //
//     //                         D o c k e r   B u i l d s
//     // ========================================================================== //
//
//     stage('Build Container') {
//       steps {
//         milestone(ordinal: null, label: "Milestone: Docker Build")
//         timeout(time: 60, unit: 'MINUTES') {
//           dir("$SOURCE_FOLDER") {
//        //     sh "docker build -t $DOCKER_IMAGE:$DOCKER_TAG -t $DOCKER_IMAGE:latest --build-arg=BUILDKIT_INLINE_CACHE=1 --cache-from $DOCKER_IMAGE:latest ."
//               sh "docker build -t $DOCKER_IMAGE:latest --build-arg=BUILDKIT_INLINE_CACHE=1 --cache-from $DOCKER_IMAGE:latest ."
//           }
//         }
//       }
//     }
//
//   // ========================================================================== //
//   //                               D e p l o y s
//   // ========================================================================== //
//
//     stage('Deploy') {
//       // agent { label "$DEPLOY_AGENT" }
//       steps {
//         script {
//           lock(resource: "Deploy - App: $APP_CODE, Environment: $DEPLOY_ENV", inversePrecedence: true) {
//             milestone(ordinal: null, label: "Milestone: Deploy")
//             echo 'Deploying...'
//             timeout(time: 15, unit: 'MINUTES') {
//               echo "Deploy to $DEPLOY_ENV"
//               try {
//                   sh "docker stop $SERVICE_NAME"
//                   sh "docker rm $SERVICE_NAME"
//               } catch (Exception e) {
//                   echo "An exception occurred: ${e.message}"
//               }
//               sh "docker run -d --network cms-wbv --name $SERVICE_NAME --memory=$SERVICE_MEM_LIMIT -p $SERVICE_PORT_PUBLISH:$SERVICE_PORT_LOCAL -v $LOGS_FOLDER:$LOGS_SOURCE $SERVICE_ARGS $DOCKER_IMAGE:$DOCKER_TAG"
//
//             }
//           }
//         }
//       }
//     }
//
//   // ========================================================================== //
//   //                               H e a l t h   C h e c k
//   // ========================================================================== //
//
//     stage('Health Check') {
//       // agent { label "$DEPLOY_AGENT" }
//       steps {
//         script {
//           timeout(time: 3, unit: 'MINUTES') {
//             waitUntil(initialRecurrencePeriod: 1000, quiet: true) {
//               def healthCheckResult = sh(script: "docker exec $SERVICE_NAME $HEALTH_CHECK_CMD", returnStatus: true)
//               if (healthCheckResult == 0) {
//                   return true
//               } else {
//                   return false
//               }
//             }
//           }
//         }
//       }
//     }
//   }
//   // ========================================================================== //
//   //                                    P o s t
//   // ========================================================================== //
//
//   post {
//     always {
//       // Clean workspace
//       cleanWs()
//       echo 'Always'
//     }
//     success {
//       echo 'SUCCESS!'
//     }
//     fixed {
//       echo "FIXED!"
//     }
//     failure {
//       echo 'FAILURE!'
//     }
//   }
// }
pipeline {
    agent {label 'slave'}

    parameters {
        booleanParam(name: 'RELEASE', defaultValue: false, description: 'Is this a Release Candidate?')
    }

    environment {
        RELEASE_VERSION = '1.1.0'
        INT_VERSION = 'R2'
    }
    stages {
        stage('Audit tools') {
            steps {
                sh '''
                  git version
                  java -version
                  mvn -version
                '''
            }
        }

        stage('Unit Test') {
            steps {

                sh '''
                    echo "Executing Unit Tests..."
                    mvn test
                '''

            }
        }

        stage('Build') {
            environment {
                VERSION_SUFFIX = "${sh(script:'if [ "${RELEASE}" = false ] ; then echo -n "${INT_VERSION}"ci:"${BUILD_NUMBER}"; else echo -n "${RELEASE_VERSION}":"${BUILD_NUMBER}"; fi', returnStdout: true)}"
            }
            steps {
                echo "Building version: ${INT_VERSION} with suffix: ${VERSION_SUFFIX}"
                echo 'Mention your Application Build Code here!!!'

                sh '''
                    mvn versions:set -DnewVersion="${VERSION_SUFFIX}"-SNAPSHOT
                    mvn versions:update-child-modules
                    mvn clean package
                '''

            }
        }

        stage('Publish') {
            when {
                expression { return params.RELEASE }
            }

            steps {
                archiveArtifacts('**/*.jar')
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
