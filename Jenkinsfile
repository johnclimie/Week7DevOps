pipeline {
    agent any

    environment {
        ImageRegistry = 'johnclimie'
        EC2_IP = '18.119.119.255'
        DockerComposeFile = 'docker-compose.yml'
        DotEnvFile = '.env'
    }

    stages {
        stage("buildImage") {
            steps {
                script {
                    echo "Building Docker Image..."
                    bat 'docker build -t %ImageRegistry%/%JOB_NAME%:%BUILD_NUMBER% .'
                }
            }
        }

        stage("pushImage") {
            steps {
                script {
                    echo "Pushing Image to DockerHub..."
                    withCredentials([usernamePassword(credentialsId: 'docker-login', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        bat """
                        echo %PASS% | docker login -u %USER% --password-stdin
                        docker push %ImageRegistry%/%JOB_NAME%:%BUILD_NUMBER%
                        """
                    }
                }
            }
        }

        stage("deployCompose") {
            steps {
                script {
                    echo "Deploying with Docker Compose..."
                    sshagent(['ec2']) {
                        // Use double quotes to allow variable expansion in bat, and escape inner quotes
                        bat """
                        scp -o StrictHostKeyChecking=no %DotEnvFile% %DockerComposeFile% ubuntu@%EC2_IP%:/home/ubuntu
                        ssh -o StrictHostKeyChecking=no ubuntu@%EC2_IP% "docker compose -f /home/ubuntu/%DockerComposeFile% --env-file /home/ubuntu/%DotEnvFile% down"
                        ssh -o StrictHostKeyChecking=no ubuntu@%EC2_IP% "docker compose -f /home/ubuntu/%DockerComposeFile% --env-file /home/ubuntu/%DotEnvFile% up -d"
                        """
                    }
                }
            }
        }
    }
}
