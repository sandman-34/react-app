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
                    // 1. Build the image with the correct username
                    sh 'docker build -t sandman34/react-app:latest .'

                    // 2. Run the container for testing
                    sh 'docker run -d -p 1233:80 --name my-test-app sandman34/react-app:latest'
            
                    sh 'sleep 5'
                    sh 'curl http://localhost:1233'
            
                    sh 'docker stop my-test-app && docker rm my-test-app'
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_login', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    // This will now use sandman34
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                
                    // Push the image we built in the previous stage
                    sh "docker push sandman34/react-app:latest"
                
                    sh "docker logout"
                    }
                }
            }
        }
         stage('DeployToStaging') {
            when {
                branch 'main'
            }
            steps {
                    script {
                        sh "docker pull sandman-34/react-app:${env.BUILD_NUMBER}"
                        try {
                            sh "docker stop react-app"
                            sh "docker rm react-app"
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "docker run --restart always --name react-app -p 1233:80 -d sandman-34/react-app:${env.BUILD_NUMBER}"
                    }
            }
        }
        
        stage("Check HTTP Response") {
            steps {
                script {
                    final String url = "http://localhost:1233"
                    
                    final String response = sh(script: "curl -o /dev/null -s -w '%{http_code}\\n' $url", returnStdout: true).trim()
                    
                    if (response == "200") {
                        echo response
                        println "Successful Response Code" 
                    } else {
                        echo response
                        println "Error Response Code" 
                    }

                }
            }
        }
        
        stage('DeployToProduction') {
            when {
                branch 'main'
            }
            steps {
                input 'Does the staging environment look OK? Did You get 200 response?'
                 milestone(1)
                    script {
                        sh "docker pull sandman-34/react-app:${env.BUILD_NUMBER}"
                        try {
                            sh "docker stop react-app"
                            sh "docker rm react-app"
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "docker run --restart always --name react-app -p 1233:80 -d sandman-34/react-app:${env.BUILD_NUMBER}"
                    }
            }
        }
    }
}
