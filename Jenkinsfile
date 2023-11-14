pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Clean workspace ') {
            steps {
                cleanWs()
            }
        }
        
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/MSFaizi/Amazon_Clone_TF.git'
            }
        }
        
        stage('Sonarqube Code Analysis') {
            steps {
                script{
                    withSonarQubeEnv('sonar-server') {
                        sh ''' 
                        $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Amazon \
                        -Dsonar.projectKey=Amazon 
                        '''
         
                    }
                }
            }
        }
        
        stage('Sonarqube Quality Gate') {
            steps {
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        
        stage('NPM') {
            steps {
               sh 'npm install '
            }
        }
        
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        
        stage('Docker Build and docker push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                         sh '''
                         docker build -t amazon .
                         docker tag amazon shamimfaizi/amazon:latest
                         docker push shamimfaizi/amazon:latest '''
                   }
                }
            }
        }
        
        stage('TRIVY docker image scan') {
            steps {
                sh "trivy image shamimfaizi/amazon:latest > trivyImage.txt"
            }
        }
        
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name amazon -p 3000:3000 shamimfaizi/amazon:latest'
            }
        }
    }
}
