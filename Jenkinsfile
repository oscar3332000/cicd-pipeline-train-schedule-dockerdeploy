pipeline {
    agent any
    
    stages {
     
        stage ("Building project") {
            
            steps {
                echo "building project"
                sh "./gradlew build --no-daemon"
                archiveArtifacts artifacts: "dist/trainSchedule.zip"
            }
            
        }
        
        stage ("build docker image") {
            when {
                branch "master"
            }
            steps {
                echo "building image"
                script {
                    app = docker.build("oscar3332000/train-schedule")
                    app.inside {
                       sh 'echo $(curl localhost:8080)'
                    }
             } //script
            } //steps
            
        } //stage
        
        stage ("push the image to dockerhub") {
            when {
             branch "master"   
            }
            
            steps {
                echo "pushing the image to docker hub"
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            } //steps
        } //stage
        
        
        stage ("Deploy to production") {
            when {
             branch "master"   
            }
            steps {
                input "Ready to deploy to production ?"
                milestone(1)
                echo "Deploying project to production"
                withCredentials([usernamePassword(credentialsId: 'webserver_login', passwordVariable: 'USERPASS', usernameVariable: 'USERNAME')]) {
                    script {
                        def remote = [:]
                        remote.name = 'production'
                        remote.host = '$prod_ip'
                        remote.user = '$USERNAME'
                        remote.password = '$USERPASS'
                        remote.allowAnyHosts = true
                    }
                    
                    sshCommand remote: remote, command: "docker pull oscar3332000/train-schedule:${env.BUILD_NUMBER}"
                    sshCommand remote: remote, command: "docker stop oscar3332000/train-schedule"
                    sshCommand remote: remote, command: "docker rm oscar3332000/train-schedule"
                    sshCommand remote: remote, command: "docker run --restart always --name train-schedule -p 8080:8080 -d oscar3332000/train-schedule:${env.BUILD_NUMBER}"
                } //with credentials
            } //steps
        } //stage
       
    }
    
}
