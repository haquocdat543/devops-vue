pipeline{
    agent { label 'Jenkin-Agent' }
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar'
        GIT_REPO_NAME = "devops-argocd"
        GIT_USER_NAME = "haquocdat543"
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/haquocdat543/devops-vue.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Vue1.0 \
                    -Dsonar.projectKey=Vue1.0 '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar' 
                }
            } 
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t vuev1 ."
                       sh "docker tag vuev1 haquocdat543/vuev1:latest "
                       sh "docker push haquocdat543/vuev1:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image haquocdat543/vuev1:latest > trivyimage.txt" 
            }
        }
	stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/haquocdat543/devops-argocd.git'
            }
        }
        stage('Update Deployment File') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                       NEW_IMAGE_NAME = "haquocdat543/vuev1:latest"
                       sh "sed -i 's|image: .*|image: $NEW_IMAGE_NAME|' resources/deployment.yaml"
                       sh 'git config --global user.name "haquocdat543"'
                       sh 'git config --global user.email "wwwdatha543@gmail.com"'
                       sh 'git add resources/deployment.yaml'
                       sh "git commit -m 'Update deployment image to $NEW_IMAGE_NAME'"
                       sh "git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main"
                    }
                }
            }
        }
    }
}
