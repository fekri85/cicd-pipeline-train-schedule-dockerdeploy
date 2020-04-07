pipeline {
    agent any
    stages {
        stage('build') {
            steps {
                echo 'Running build automation'
                sh './gradlem build —no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image) {
              when {
                  branch 'master' 
              }
              steps {
                  script {
                      app = docker.build("yaserfekri/firstimage") 
                      app.inside {
                       sh 'echo $(curl localhost:8080)'
                      }
                  }
              }
              }
              stage('Posh docker Image') {
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
              stage'DeployToProduction' {
                  when {
                      branch 'master' 
                  }
                  steps {
                      input 'Deploy to Production?' 
                          milestone(1)
                      withCredentials([usernamePassword(credentialsId: 'localwebserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                                                        script {
                                                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeythecking=no $USERNAME@$prod_ip \"docker pull yaserfekri/firstimage:${env.BUILD_NUMBER}\""
                                                            try {
                                                                sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeythecking=no $USERNAME@$prod_ip \"docker stop firstimage\""
                                                                sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeythecking=no $USERNAME@$prod_ip \"docker rm firstimage\""
                                                            } catch (err) {
                                                            echo: 'caught error: $err' 
                                                            }
                                                             sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeythecking=no $USERNAME@$prod_ip \"docker run  --restart always -name firstimage -p 8081:8080 -d yaserfekri/firstimage:${env.BUILD_NUMBER}\""
                  }
             }
       }
  }
}
                                                        
