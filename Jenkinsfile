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
                    // 1. Build the image
                    sh 'docker build -t sandman-34/react-app:latest .'

                    // 2. Run the container with port mapping (Host 1233 -> Container 80)
                    // We use --entrypoint='' to ensure Jenkins can interact with it if needed
                    sh 'docker run -d -p 1233:80 --name my-test-app sandman-34/react-app:latest'
                    
                    // 3. Wait for Nginx to wake up and test
                    sh 'sleep 5'
                    sh 'curl http://localhost:1233'
                    
                    // 4. Cleanup test container
                    sh 'docker stop my-test-app && docker rm my-test-app'
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    // This uses the Global Credential 'dockerhub_login' you just created
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub_login') {
                        sh 'docker push sandman-34/react-app:latest'
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
