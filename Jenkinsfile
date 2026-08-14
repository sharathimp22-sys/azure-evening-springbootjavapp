pipeline {

    agent any

    tools {

        maven 'Maven3'

    }

    environment {
        TENANT_ID="ec78375d-0db0-42cf-82a6-2e6403e95936"
        IMAGE_NAME = "sprinbootapp"
        IMAGE_TAG = "latest"
        ACR_LOGIN_SERVER = "project4springboot.azurecr.io"
        
    }

    stages {
        stage('Check Out from Git') 
        {
            steps {
                git branch: 'prod' , url: 'https://github.com/sharathimp22-sys/azure-evening-springbootjavapp.git'
            }
        }

        stage('Maven Validate') 
        {
             steps {
                 sh 'mvn validate'
             }
        }

            stage('Maven Compile') 
             {
                 steps {
                     sh 'mvn compile'
                 }
             }
         stage('Maven Test') 
         {
             steps {
                sh 'mvn test'
             }
         }
         stage('Maven Install') 
         {
             steps {
                 sh 'mvn install'
             }
         }
        stage(' Trivy Scan')
        {
            steps {
                echo "Trivy Scan Started"
                sh 'trivy fs --scanners vuln --format table --output trivy-report.txt --severity HIGH,CRITICAL .'
                archiveArtifacts artifacts: 'trivy-report.txt', fingerprint: true
                echo "Trivy Scan Finished"
            }
        }

        stage('Sonar Analysis') {
    steps {
        echo 'SonarQube Analysis Started'

        withSonarQubeEnv('SonarQube') {
            sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=azure-evening-springbootjavapp'
        }

        echo 'SonarQube Analysis Finished'
    }
}
        
        stage('Maven Package') 
        {
            steps {
                sh 'mvn package'
            }
        }
        stage('Sonar Quality Gate') 
        {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
                echo "Sonar Quality Gate Finished"
            }
        }
      }
      stage('Docker Build') {
    steps {
        echo "Build Docker Image"
        sh 'docker build -t "${IMAGE_NAME}:${IMAGE_TAG}" .'
    }
}

stage('Trivy Image Scan') {
    steps {
        echo 'Scanning Docker image'

        sh '''
            trivy image \
            --severity HIGH,CRITICAL \
            --format table \
            --output trivy-image-report.txt \
            "${IMAGE_NAME}:${IMAGE_TAG}"
        '''

        archiveArtifacts artifacts: 'trivy-image-report.txt', fingerprint: true

        echo 'Trivy Image Scan Finished'
    }
}

stage('Push to ACR') {
    steps {
        echo 'Pushing Docker image to Azure Container Registry'

        withCredentials([
            usernamePassword(
                credentialsId: 'acr-creds',
                usernameVariable: 'ACR_USER',
                passwordVariable: 'ACR_PASS'
            )
        ]) {
            sh '''
                echo "$ACR_PASS" | docker login "$ACR_LOGIN_SERVER" \
                    -u "$ACR_USER" \
                    --password-stdin

                docker tag \
                    "${IMAGE_NAME}:${IMAGE_TAG}" \
                    "${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"

                docker push \
                    "${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"

                docker logout "$ACR_LOGIN_SERVER"
            '''
        }

        echo 'Docker image pushed to ACR successfully'
    }
}
    }
}
