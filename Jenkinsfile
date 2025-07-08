pipeline {
    agent any

    environment {
        ImageRegistry = 'johnclimie'
        EC2_IP = '18.119.119.255'
        DockerComposeFile = 'docker-compose.yml'
        DotEnvFile = '.env'
        SSH_KEY_PATH = 'C:\\Users\\John Climie\\Desktop\\ssh\\ec2.pem'  // Update this to your actual private key path on Jenkins node
    }

    stages {
        stage("buildImage") {
            steps {
                script {
                    echo "Building Docker Image..."
                    bat "docker build -t %ImageRegistry%/%JOB_NAME%:%BUILD_NUMBER% ."
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

        stage('Deploy with Docker Compose') {
            steps {
                echo 'Deploying with Docker Compose...'
                bat """
                scp -o StrictHostKeyChecking=no -i C:\\Users\\JohnCl~1\\Desktop\\ssh\\ec2.pem .env docker-compose.yml ubuntu@18.119.119.255:/home/ubuntu
                ssh -o StrictHostKeyChecking=no -i C:\\Users\\JohnCl~1\\Desktop\\ssh\\ec2.pem ubuntu@18.119.119.255 "docker compose -f /home/ubuntu/docker-compose.yml --env-file /home/ubuntu/.env down"
                ssh -o StrictHostKeyChecking=no -i C:\\Users\\JohnCl~1\\Desktop\\ssh\\ec2.pem ubuntu@18.119.119.255 "docker compose -f /home/ubuntu/docker-compose.yml --env-file /home/ubuntu/.env up -d"
                """
            }
        }


    }
}
