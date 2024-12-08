pipeline {
    agent any

    tools {
        maven 'my-maven' // Đảm bảo tên này khớp với cấu hình trong Jenkins
    }

    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root-login') // Thiết lập credentials với ID 'mysql-root-login' trong Jenkins
    }

    stages {
        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Packaging/Pushing image') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') { // Đảm bảo bạn có credentials với ID 'dockerhub' trong Jenkins
                    sh 'docker build -t tinlt/springboot .' // Cập nhật tên image nếu cần
                    sh 'docker push tinlt/springboot' // Cập nhật tên image nếu cần
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop khalid-mysql || echo "this container does not exist"'
                sh 'docker volume rm khalid-mysql-data || echo "no volume"'

                sh '''
                   docker run -d --name khalid-mysql \
                   --network dev \
                   -v khalid-mysql-data:/var/lib/mysql \
                   -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} \
                   -e MYSQL_DATABASE=db_example \
                   mysql:8.0
                '''
                sh 'sleep 20'
                sh '''
                   docker exec -i khalid-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script.sql
                '''
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull tinlt/springboot' // Cập nhật tên image nếu cần
                sh 'docker container stop khalid-springboot || echo "this container does not exist"'
                sh 'docker network create dev || echo "this network exists"'

                sh '''
                    docker container run -d --rm --name khalid-springboot \
                    -p 8081:8080 \
                    --network dev \
                    tinlt/springboot // Cập nhật tên image nếu cần
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
