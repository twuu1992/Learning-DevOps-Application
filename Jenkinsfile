pipeline{
    agent any
    environment {
        APP_SERVER_NAME = "my-user-app"
        DB_SERVER_NAME = "my-user-db"
    }
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
                sh '''
                cd Learning-DevOps-Application

                echo 'Replace the variables of all env files'
                unset MONGODB_IP
                unset API_URI
                APP_SERVER_IP=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=$APP_SERVER_NAME" \
                 --query 'Reservations[*].Instances[*].[PublicIpAddress]' --output text)
                MONGODB_IP=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=$DB_SERVER_NAME" \
                --query 'Reservations[*].Instances[*].[PublicIpAddress]' --output text)
                API_URI=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=$APP_SERVER_NAME" \
                --query 'Reservations[*].Instances[*].[PublicDnsName]' --output text)
                
                sed -i -e "s/{mongodb_ip}/$MONGODB_IP/g" ./.env
                sed -i -e "s/{api_uri}/$API_URI/g" ./client/src/environments/environment.prod.ts

                echo 'Copy files to remote server'
                if ! ssh jenkins@${APP_SERVER_IP} '[ -d /home/jenkins/docker-deployment ]'; then
                    ssh jenkins@${APP_SERVER_IP} 'mkdir /home/jenkins/docker-deployment'
                fi

                scp ./docker-compose.yml jenkins@${APP_SERVER_IP}:/home/jenkins/docker-deployment/docker-compose.yml
                scp ./.env jenkins@${APP_SERVER_IP}:/home/jenkins/docker-deployment/.env
                '''
            }
        }
        stage('Deploy Docker-Compose Remotely'){
            steps{
                sh '''
                APP_SERVER_IP=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=$APP_SERVER_NAME" \
                 --query 'Reservations[*].Instances[*].[PublicIpAddress]' --output text)
                ssh jenkins@${APP_SERVER_IP}
                cd /home/jenkins/docker-deployment
                sudo chmod +x docker-compose.yml
                sudo docker compose pull
                sudo docker compose --env-file=.env up -d
                '''
            }
        }
        // stage('Clean After Deployment'){
        //     steps{
        //         sh '''
        //         echo 'Clean Directory'
        //         rm -rf cd Learning-DevOps-Application
        //         '''
        //     }
        // }
    }
}