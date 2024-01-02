pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/IshwarKhadka000/Netflix-clone.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install --legacy-peer-deps --force"
            }
        }
        stage("Docker Build & Push"){
            steps{
                sh "docker build --build-arg TMDB_V3_API_KEY=3b453f64a5e859da329ab651a9ec24c9 -t netflix ."
                sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                sh "docker tag netflix ishwarkhadka/netflix:latest"
                sh "docker push ishwarkhadka/netflix:latest"
                
            }
        }
        
        stage('Deploy to container'){
            steps{
                sh 'docker run -d -p 8081:80 ishwarkhadka/netflix:latest'
                
            }
        }

    }
    
}