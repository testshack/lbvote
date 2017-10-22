pipeline {
    agent { label 'ubuntu' }

    options {
        buildDiscarder(logRotator(numToKeepStr: '30', artifactNumToKeepStr: '30'))
        timestamps()
        ansiColor('xterm')
    }

    environment {
        APP_DATACENTER = ''
        APP_MAIN = pwd()
		GIT_URL = 'https://github.com/testshack/joara-main'
		GIT_CREDENTIALS = 'jenkins_github_credentials'
		GIT_BRANCH = 'master'
    }

    stages {
       stage('Check Preconditions') {
          when {
              expression {
                   env.BRANCH_NAME != 'dev' && env.BRANCH_NAME != 'test' && env.BRANCH_NAME != 'master'
              }
          }
          steps {
              script {
                  currentBuild.result = 'ABORTED'
              }
              error('Image is built only against dev,test and master branch')
          }
        }

        stage('Get APP') {
            steps {
                script {

                        checkout( [$class: 'GitSCM',
                            branches: [[name: '*/master' ]],
                            userRemoteConfigs: [[
                                credentialsId: 'jenkins_github_credentials',
                                url: 'https://github.com/testshack/joara-main']]
                            ]
                        )

                      }

            }


        }

        stage('Get Image To Build') {
           steps {
               script {

                dir('infrastructure/images/lbvote') {
                       git branch: env.BRANCH_NAME, credentialsId: 'jenkins_github_credentials', url: 'https://github.com/testshack/lbvote.git'
                     }
                   }
           }


               }

         stage('Install APP') {
            steps {
                sh 'ls -la infrastructure/images'
                sh 'ls -la infrastructure/images/lbvote/'
            }
         }
         stage('Get SSH Keys') {
             steps {
                withCredentials([[$class: 'FileBinding',credentialsId: 'acs_private_key', variable: 'ENV_SSH_FILE']]) {

                     sh 'mkdir -p ~/.ssh'
                     sh 'rm -rf ~/.ssh/id_rsa'
                     sh 'cp $ENV_SSH_FILE ~/.ssh/id_rsa'
                     sh 'chmod 0600 ~/.ssh/id_rsa'
                 }

             }
          }
           stage('Install APP and manage docker images') {
            steps {
                withCredentials([azureServicePrincipal("jenkins_azure_credentials_${env.BRANCH_NAME}")]) {

                        sh 'chmod +x build-scripts.sh'
                        sh './build-scripts.sh $BRANCH_NAME lbvote'

                        }
            }


                }

    }

   post {
             always {
                echo 'Job finished'
                deleteDir() /* clean up our workspace */
            }
            success {
                echo 'Job succeeeded!'
                mail to: 'nullfire42@hotmail.com',
                subject: "Success Pipeline: ${currentBuild.fullDisplayName}",
                body: "Build completed successfully with ${env.BUILD_URL}"
            }
            unstable {
                echo 'Job unstable :/'
            }
            changed {
                echo 'Things were different before...'
            }
            failure {
             echo 'Job failed :('
             mail to: 'nullfire42@hotmail.com',
             subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
             body: "Something is wrong with ${env.BUILD_URL}"
             }
        }
}