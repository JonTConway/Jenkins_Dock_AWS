pipeline {
    agent any

    environment {
        ImageRegistry = 'jconway87'
        EC2_IP = '44.195.49.46'
        DockerComposeFile = 'docker-compose.yml'
        DotEnvFile = '.env'
    }

    stages {
        stage("buildImage") {
            steps {
                script {
                    echo "Building Docker Image..."
                    // Uses Windows bat to build the image locally
                    bat "docker build -t ${ImageRegistry}/${JOB_NAME}:${BUILD_NUMBER} ."
                }
            }
        }

       stage("pushImage") {
            steps {
                script {
                    echo "Pushing Image to DockerHub..."
                    withCredentials([usernamePassword(credentialsId: 'docker-login', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        // This native docker flag avoids using 'echo' and 'pipe' entirely
                        bat 'docker login -u %USER% -p %PASS%'
                        bat "docker push ${ImageRegistry}/${JOB_NAME}:${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage("deployCompose") {
            steps {
                script {
                    echo "Deploying with Docker Compose to Ubuntu EC2..."
                    withCredentials([sshUserPrivateKey(credentialsId: 'ec2', keyFileVariable: 'SSH_KEY')]) {
                        sh """
                        # 1. Start the native shell agent manually
                        eval \$(ssh-agent -s)
                        
                        # 2. Add the temporary key file injected by Jenkins
                        ssh-add "\${SSH_KEY}"
                        
                        # 3. Your original deployment commands
                        scp -o StrictHostKeyChecking=no ${DotEnvFile} ${DockerComposeFile} ubuntu@${EC2_IP}:/home/ubuntu
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} "docker compose -f /home/ubuntu/${DockerComposeFile} --env-file /home/ubuntu/${DotEnvFile} down"
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} "docker compose -f /home/ubuntu/${DockerComposeFile} --env-file /home/ubuntu/${DotEnvFile} up -d"
                        """
                    }
                }
            }
        }
    }
}