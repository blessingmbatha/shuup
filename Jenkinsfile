pipeline {
  agent any
        environment {
              SCANNER_HOME=tool 'sonar-scanner'
        }

     stages {
         stage('Cleanup Workspace') {
             steps {
                cleanWs ()
             }
         }
         stage('Git Checkout') {
             steps {
                git changelog: false, poll: false, url: 'https://github.com/blessingmbatha/shuup.git'
             }
         }
         stage('Install  Dependencies') {
             steps {
                sh "pip install -r requirements.txt"
             }
         }
         stage('Run Tests') {
             steps {
                sh 'python3 -m unittest discover'
             }
         }
         stage('Build') {
             steps {
                sh 'python3 setup.py build'
             }
         }
         stage("Sonarqube Analysis "){
             steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=shuup \
                    -Dsonar.projectKey=shuup '''
                }
            }
         }
          stage("OWASP Dependency Check"){
              steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
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
                   withDockerRegistry(credentialsId: 'docker-cred2', toolName: 'docker'){   
                       sh "docker build -t shuup:lastest ."
                       sh "docker tag shuup nkosenhlembatha/shuup:latest "
                       sh "docker push nkosenhlembatha/shuup:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image nkosenhlembatha/shuup:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name shuup -p 8081:80 nkosenhlembatha/shuup:latest'
            }
        }
    }
}
