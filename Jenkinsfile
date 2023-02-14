pipeline{
    agent any
    def APP_SERVER_IP = ""
    stages {
        stage('Docker Version and Login') {
            sh '''
            docker --version
            docker compose version
            docker login -u="$DOCKER_USERNAME" -p="$DOCKER_PASSWORD"
            '''
        }

        stage('Checkout Repo') {
            steps{
                cleanWs()
                sh 'git clone https://github.com/twuu1992/Learning-DevOps-Application.git'
            }
        }
        stage('Build And Push Backend Image'){
            steps{
                sh '''
                cd Learning-DevOps-Application
                echo 'Build user server app'
                cd server
                docker build -t ${DOCKER_USERNAME}/user-server:latest .
                docker push ${DOCKER_USERNAME}/user-server:latest
                '''
            }
        }
        stage('Build And Push Client Image'){
            steps{
                sh '''
                cd Learning-DevOps-Application
                echo 'Build user client app'
                cd client
                docker build -t ${DOCKER_USERNAME}/user-client:latest .
                docker push ${DOCKER_USERNAME}/user-client:latest
                '''
            }
        }
        stage('Build And Push NGINX Image'){
            steps{
                sh '''
                cd Learning-DevOps-Application
                echo 'Build nginx service'
                cd nginx
                docker build -t ${DOCKER_USERNAME}/user-nginx:latest .
                docker push ${DOCKER_USERNAME}/user-nginx:latest
                '''
            }
        }
        stage('Clean after Docker Build'){
            steps{
                sh '''
                echo 'Clean docker'
                docker system prune -f
                '''
            }
        }
        stage('Prepare for Docker-Compose Deployment'){
            steps{
                script{
                    APP_SERVER_IP = sh(script: "aws ec2 describe-instances --filters ""Name=tag:Name,Values=my-user-app"" \
                --query 'Reservations[*].Instances[*].[PublicIpAddress]' --output text",
                returnStdout: true)
                }
                sh '''
                cd Learning-DevOps-Application

                echo 'Replace the mongo db ip address to env file'
                unset MONGODB_IP
                MONGODB_IP=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=my-user-db" \
                --query 'Reservations[*].Instances[*].[PublicIpAddress]' --output text)
                echo "MONGODB SERVER IP: $MONGODB_IP
                sed -i -e "s/{mongodb_ip}/$MONGODB_IP}/g" ./.env

                echo 'Copy files to remote server'
                scp ./docker-compose.yml ubuntu@${APP_SERVER_IP}:~/docker-deployment
                scp ./.env ubuntu@${APP_SERVER_IP}:~/docker-deployment
                '''
            }
        }
        stage('Deploy Docker-Compose Remotely'){
            steps{
                sh '''
                ssh ubuntu@${APP_SERVER_IP}
                cd ~/docker-deployment
                sudo chmod +x docker-compose.yml
                sudo docker compose pull
                sudo docker compose --env-file=.env up -d

                exit
                '''
            }
        }
        stage('Clean After Deployment'){
            steps{
                sh '''
                echo 'Clean Directory'
                rm -rf cd Learning-DevOps-Application
                '''
            }
        }
    }
}