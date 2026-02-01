pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh 'chmod +x gradlew'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/reactApp'
            }
        }
        stage('Docker Build and Test') {
            steps {
                script {
                    // This line is the 'insurance policy' to prevent port conflicts
                    sh 'docker stop my-test-app || true && docker rm my-test-app || true'
            
                    sh 'docker build -t sandman34/react-app:latest .'
                    sh 'docker run -d -p 1233:80 --name my-test-app sandman34/react-app:latest'
                    sh 'sleep 5'
                    sh 'curl http://localhost:1233'
            
                    // If you want to see the 'Blue Box' later, 
                    // keep the stop/rm commands at the end so it stays clean!
                    sh 'docker stop my-test-app'
                    sh 'docker rm my-test-app'
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_login', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                
                    // Tag it with BOTH latest and the build number
                    sh "docker tag sandman34/react-app:latest sandman34/react-app:${env.BUILD_NUMBER}"
                
                    sh "docker push sandman34/react-app:latest"
                    sh "docker push sandman34/react-app:${env.BUILD_NUMBER}"
                
                    sh "docker logout"
                    }
                }
            }
        }
        stage('DeployToStaging') {
            steps {
                script {
                    sh "docker stop staging-app || true && docker rm staging-app || true"
                    // Pulling with build number to ensure we get the fresh image
                    sh "docker pull sandman34/react-app:${env.BUILD_NUMBER}"
                    sh "docker run -d -p 8081:80 --name staging-app sandman34/react-app:${env.BUILD_NUMBER}"
                }
            }
        }
        stage('Check HTTP Response') {
            steps {
                script {
                    echo "Checking Staging App on port 8081..."
                    sh "sleep 10" 
                    sh "curl http://localhost:8081"
                }
            }
        }
        stage('DeployToProduction') {
            // This is the magic part that creates the blue box
            input {
                message "Does the staging environment look OK? Did you get 200 response?"
                ok "Proceed"
            }
            steps {
                script {
                    sh "docker stop production-app || true && docker rm production-app || true"
                    sh "docker run -d -p 80:80 --name production-app sandman34/react-app:latest"
                }
            }
        }
    }
}
