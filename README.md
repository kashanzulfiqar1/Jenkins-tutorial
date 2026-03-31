# Jenkins-tutorial</br>
Jenkins tutorial document ****</br>
**Sample pipeline**</br>
"pipeline {</br>
    agent any</br>
</br>
    stages {</br>
        stage('Git Login and Pull') {</br>
            steps {</br>
                sh 'rm -rf testproject' </br>
                sh 'git clone http://172.31.191.133/root/testproject.git'</br>
            }</br>
        } </br>
</br>
        stage('docker compose installation') {</br>
            steps {</br>
                sh '''</br>
                mkdir -p ~/.docker/cli-plugins/</br>
                curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose</br>
                chmod +x ~/.docker/cli-plugins/docker-compose</br>
                '''</br>
            }</br>
        }</br>
</br>
        stage('Make build using docker build command') {</br>
            steps {</br>
                sh '''</br>
                cd testproject</br>
                # Building WITHOUT a tag defaults to 'latest'</br>
                docker build -t kashan1234/testimagefromjenkins .</br>
                docker images -a</br>
                '''</br>
            }</br>
        }</br>
</br>
        stage('docker login and push image on docker hub') {</br>
            steps {</br>
                script {</br>
                    // Match the name exactly as built in the previous stage</br>
                    def myImage = docker.image("kashan1234/testimagefromjenkins")</br>
                    </br>
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-creds') {</br>
                        myImage.push('latest')</br>
                    }</br>
                }</br>
            }</br>
        }</br>
        stage('deployment') {</br>
            steps {</br>
                sshagent(['vm-ssh-key']) {</br>
            sh '''</br>
            ssh -o StrictHostKeyChecking=no ubuntu@3.88.224.255 << 'EOF'</br>
sudo apt-get update -y</br>
if ! command -v docker &> /dev/null; then</br>
    echo "Installing Docker..."</br>
    sudo apt-get install -y docker.io</br>
    sudo systemctl start docker</br>
    sudo usermod -aG docker ubuntu</br>
    </br>
    mkdir -p ~/.docker/cli-plugins/</br>
    curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose</br>
    chmod +x ~/.docker/cli-plugins/docker-compose</br>
else</br>
    echo "Docker is already installed."</br>
fi</br>
docker pull kashan1234/testimagefromjenkins:latest</br>
docker run -it -d -p 80:80 --name newcont1 kashan1234/testimagefromjenkins:latest</br>
EOF</br>
'''</br>
                }</br>
            }</br>
        }</br>
    } </br>
</br>
    post {</br>
        always {</br>
            cleanWs() </br>
        }</br>
    }</br>
} "</br>
</br>
