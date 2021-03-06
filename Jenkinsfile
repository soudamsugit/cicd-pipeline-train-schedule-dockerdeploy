pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('docker build') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("sureshsouven/trainschedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage ('docker push') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login'){
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Docker Deploy') {
            when {
                branch 'master'
            }
            steps {
                input 'deploy to production?'
                milestone (1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]){
                    script {
                        sh "sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull sureshsouven/trainschedule:${env.BUILD_NUMBER}\""
                        try {
                          sh "sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop trainschedule\""
                          sh "sshpass -p $USERPASS -v ssh -o StrictHostkeyChecking=no $USERNAME@$prod_ip \"docker rm trainschedule\""
                        }
                        catch (err) {
                            'caught error: $err'
                        }
                        sh "sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d  sureshsouven/trainschedule:${env.BUILD_NUMBER}\"" 
                    }
                }
            }
        }
    }
}
