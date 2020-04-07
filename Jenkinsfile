pipeline {
    agent any
    stages {
        stage('build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts:--no-daemon 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
              when {
                  branch 'master' 
              }
              steps {
                  script {
                      app = docker.build("yaserfekri/train-schedule") 
                      app.inside {
                       sh 'echo $(curl localhost:8080)'
                      }
                  }
              }
          }
              stage('Push docker Image') {
                  when {
                      branch 'master' 
                  }
                  steps {
                      script {
                          docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                               app.push("${env.BUILD_NUMBER}")
                               app.push("latest")
                          }
                      }
                  }
              }
              stage ('DeployToProduction') {
                  when {
                      branch 'master' 
                  }
                  steps {
                      input 'Deploy to Production?' 
                          milestone(1)
                      withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                                                        script {
                                                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeychecking=no $USERNAME@$prod_ip \"docker pull yaserfekri/train-schedule:${env.BUILD_NUMBER}\""
                                                            try {
                                                                sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeychecking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
                                                                sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeychecking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
                                                            } catch (err) {
                                                            echo: 'caught error: $err' 
                                                            }
                                                             sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeychecking=no $USERNAME@$prod_ip \"docker run  --restart always -name train-schedule -p 8081:8080 -d yaserfekri/train-schedule:${env.BUILD_NUMBER}\""
                                              }
                                         }
                  }
              }
    }
}
