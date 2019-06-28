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
        
        stage ('build docker image') {
            
            when {
             branch 'master'   
            }
            steps{
                
                echo 'Building docker image'
                script {
                    app = docker.build('oscar3332000/train-schedule')
                    app.inside {
                        sh 'echo $( curl localhost:8080 )'
                    }
                }
                
            }//steps
        }//stage
        
        stage('Push Docker Image') {
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
        
        stage('docker to production') {
            when {
             branch 'master'   
            }
            
            steps { 
                input 'Deploy to production?'
                milestone(1)
                
                withCredentials([usernamePassword(credentialsId: 'production', passwordVariable: 'USERPASS', usernameVariable: 'USERNAME')]) {

                    script {
                        
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \" docker pull oscar3332000/train-schedule:${env.BUILD_NUMBER} \""
                        try {
                         sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop oscar3332000/train-schedule\""
                         sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm oscar3332000/train-schedule\""
                            
                        } catch(err)
                        {
                            echo "caught error: $err"
                        }
                        
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart-always --name train-schedule -p 8080:8080 -d oscar3332000/train-schedule:${env.BUILD_NUMBER}  \""
                    }
                    
                } //withCredentials

                
                
            } //steps
        } //stage
        
        
    }
}
