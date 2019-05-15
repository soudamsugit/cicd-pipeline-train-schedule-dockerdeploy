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
            steps {
                when 'master'
            }
            steps {
                script {
                    app=docker.build("soudamsu/trainschedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
                {
        }
        stage ('docker push') {
            steps {
                when 'master'
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
        }
        stage('Docker Deploy') {
            steps {
                when 'master'
            }
            steps {
                input 'deploy to production?'
                milestone (1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVarialbe: 'USERNAME', passwordVariable: 'PASSWORD')]){
                    script {
                        sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull soudamsu/train-schedule:$(env.BUILD_NUMBER})\""
                        try {
                          sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
                          sh "sshpass -p $PASSWORD -v ssh -o StrictHostkeyChecking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
                            
                        }
                        catch (err) {
                            'caught error: $err'
                        }
                        sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d  soudamsu/train-schedule:${env.BUILD_NUMBER}\"" 
                    }
                }
            }
        }
    }
}
