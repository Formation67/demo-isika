pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS=credentials('docker-hub-cred')
    }

    stages {
        stage('Checkout') {
            steps {
                echo '-=- Checkout project -=-'
                git branch: 'main', url:'https://github.com/Formation67/demo-isika.git'
            }
        }
        stage('Compile') {
            steps {
                echo '-=- Compile project -=-'
                sh 'mvn clean compile'
            }
        }
        stage('Test') {
            steps {
                echo '-=- Test project -=-'
                sh 'mvn test'
            }
            
            post {
                success {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Package') {
            steps {
                echo '-=- Package project -=-'
                sh 'mvn package -DskipTests'
            }
            
            post {
                always {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Cleannig') {
            steps {
                echo '-=- Clean docker images & container -=-'
                sh 'ssh -v -o StrictHostKeyChecking=no vagrant@192.168.33.20 sudo docker stop demo-isika || true'
                sh 'ssh -v -o StrictHostKeyChecking=no vagrant@192.168.33.20 sudo docker rm demo-isika || true'
                sh 'docker rmi demo-isika || true'
            }
        } 
        stage('Construct image to Staging') {
            steps {
                echo '-=- Docker build -=-'
                sh 'docker build -t demo-isika .'
            }
        }       
        stage('Tag') {
            steps {
                echo '-=- Login to dockerHub -=-'
                sh 'docker tag demo-isika doum167/demo-isika'
            }
        }       
        stage('Login') {
            steps {
                echo '-=- Login to dockerHub -=-'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Push') {
            steps {
                echo '-=- Push to dockerHub -=-'
                sh 'docker push doum167/demo-isika'
            }
        } 
        
        stage('Run container to local') {
            steps {
                echo '-=- Run container -=-'
                sh 'ssh -v -o StrictHostKeyChecking=no vagrant@192.168.33.20 sudo docker run -d --name demo-isika -p 8080:8080 doum167/demo-isika'
            }
        }
        stage ('Deploy To Prod'){
            input{
                message "Do you want to proceed for production deployment?"
            }
            steps {
                sh 'echo "Deploy into Prod"'
                sh 'ssh -v -o StrictHostKeyChecking=no ubuntu@13.38.7.90 sudo docker stop demo-isika || true'
                sh 'ssh -v -o StrictHostKeyChecking=no ubuntu@13.38.7.90 sudo docker rm demo-isika || true'
                sh 'ssh -v -o StrictHostKeyChecking=no ubuntu@13.38.7.90 sudo docker run -d --name demo-isika -p 8080:8080 doum167/demo-isika'

            }
        }
    }
    
    post {
        always {
            sh 'docker logout'
        }
    }
}