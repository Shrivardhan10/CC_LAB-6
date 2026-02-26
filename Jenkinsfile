pipeline {
    agent any

    parameters {
        choice(
            name:'BACKEND_COUNT',
            choices:['1','2'],
            description:'Number of backend containers'
        )
    }

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                docker rm -f backend1 backend2 || true

                if [ "$BACKEND_COUNT" = "1" ]; then
                    docker run -d --name backend1 backend-app
                else
                    docker run -d --name backend1 backend-app
                    docker run -d --name backend2 backend-app
                fi

                sleep 5
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true

                docker network create app-network || true

                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx

                sleep 2

                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
