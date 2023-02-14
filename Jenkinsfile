// Variables
def APP_SERVER_IP = ""

pipeline{
    agent any
    stages {
        stage('Docker Version and Login') {
            steps{
               sh '''
                docker --version
                docker compose version
                docker login -u="$DOCKER_USERNAME" -p="$DOCKER_PASSWORD"
            ''' 
            }
            
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
                echo 'Clean docker'
                docker system prune -f
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
                echo 'Clean docker'
                docker system prune -f
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
                echo 'Clean docker'
                docker system prune -f
                '''
            }
        }
        stage('Prepare for Docker-Compose Deployment'){
            steps{
                script{
                    APP_SERVER_IP = sh( script: '''aws ec2 describe-instances --filters 'Name=tag:Name,Values=my-user-app' \
                     --query 'Reservations[*].Instances[*].[PublicIpAddress]' --output text''',
                returnStdout: true).trim()
                }
                sh '''
                cd Learning-DevOps-Application

                echo 'Replace the variables of all env files'
                unset MONGODB_IP
                unset API_URI
                MONGODB_IP=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=my-user-db" \
                --query 'Reservations[*].Instances[*].[PublicIpAddress]' --output text)
                API_URI=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=my-user-app" \
                --query 'Reservations[*].Instances[*].[PublicDnsName]' --output text)
                
                sed -i -e "s/{mongodb_ip}/$MONGODB_IP}/g" ./.env
                sed -i -e "s/{api_uri}/$API_URI/g" ./client/src/environments/environment.prod.ts

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