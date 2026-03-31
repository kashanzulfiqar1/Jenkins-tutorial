# Jenkins-tutorial
Jenkins tutorial document 
**Sample pipeline**
"pipeline {
    agent any

    stages {
        stage('Git Login and Pull') {
            steps {
                sh 'rm -rf testproject' 
                sh 'git clone http://172.31.191.133/root/testproject.git'
            }
        } 

        stage('docker compose installation') {
            steps {
                sh '''
                mkdir -p ~/.docker/cli-plugins/
                curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
                chmod +x ~/.docker/cli-plugins/docker-compose
                '''
            }
        }

        stage('Make build using docker build command') {
            steps {
                sh '''
                cd testproject
                # Building WITHOUT a tag defaults to 'latest'
                docker build -t kashan1234/testimagefromjenkins .
                docker images -a
                '''
            }
        }

        stage('docker login and push image on docker hub') {
            steps {
                script {
                    // Match the name exactly as built in the previous stage
                    def myImage = docker.image("kashan1234/testimagefromjenkins")
                    
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-creds') {
                        myImage.push('latest')
                    }
                }
            }
        }
        stage('deployment') {
            steps {
                sshagent(['vm-ssh-key']) {
            sh '''
            ssh -o StrictHostKeyChecking=no ubuntu@3.88.224.255 << 'EOF'
sudo apt-get update -y
if ! command -v docker &> /dev/null; then
    echo "Installing Docker..."
    sudo apt-get install -y docker.io
    sudo systemctl start docker
    sudo usermod -aG docker ubuntu
    
    mkdir -p ~/.docker/cli-plugins/
    curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
    chmod +x ~/.docker/cli-plugins/docker-compose
else
    echo "Docker is already installed."
fi
docker pull kashan1234/testimagefromjenkins:latest
docker run -it -d -p 80:80 --name newcont1 kashan1234/testimagefromjenkins:latest
EOF
'''
                }
            }
        }
    } 

    post {
        always {
            cleanWs() 
        }
    }
} "
