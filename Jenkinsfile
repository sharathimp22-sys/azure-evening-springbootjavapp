pipeline {

    agent any

    tools {

        maven 'Maven3'

    }

    environment {
        TENANT_ID="ec78375d-0db0-42cf-82a6-2e6403e95936"
        IMAGE_NAME = "sprinbootapp"
        IMAGE_TAG = "latest"
        
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
      stage ('Docker Build')
      {
        steps {
            
            echo "Build Docker Image"
            sh  'docker build -t "${IMAGE_NAME}:${IMAGE_TAG}" .'
      
        }
      }
   }
}
