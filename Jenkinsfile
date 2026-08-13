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
        // stage('Maven Install') 
        // {
        //     steps {
        //         sh 'mvn install'
        //     }
        // }
        stage(' Trivy Scan')
        {
            steps {
                echo "Trivy Scan Started"
                sh 'trivy fs --format table --output trivy-report.txt --severity HIGH,CRITICAL .'
                echo "Trivy Scan Finished"
            }
        }

        stage('Sonar Analysis')
        {
            environment {
                SCANNER_HOME = tool 'Sonar-scanner'
            }
          steps {
              withSonarQubeEnv('sonarserver') {
                sh '''${SCANNER_HOME}/bin/sonar-scanner \
                -Dsonar.organization=bkrrajmali \
                -Dsonar.projectName=springbootapp \
                -Dsonar.projectKey=springbootapp \
                -Dsonar.java.binaries=.
                '''
              }
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
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQuality abortPipeline: true, credentialsId: 'sonar'
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
