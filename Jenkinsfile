pipeline {

    agent any

    tools {

        maven 'Maven'

    }

    environment {
        TENANT_ID="8267a341-645e-4b88-abc9-dfbc065e82d2"
        IMAGE_NAME = "sprinbootapp"
        IMAGE_TAG = "latest"
        ACR_NAME= 'springbootdockerregs'
        ACR_LOGIN_SERVER ='springbootdockerregs.azurecr.io'
        FULL_IMAGE_NAME = "${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
        RG  = 'demoaks_group'
        NAME = 'demoaks'
    }

    stages {
        stage('Check Out from Git') 
        {
            steps {
                git branch: 'prod' , url: 'https://github.com/bkrrajmali/azure-evening-springbootjavapp.git'
            }
        }

        // stage('Maven Validate') 
        // {
        //     steps {
        //         sh 'mvn validate'
        //     }
        // }

            // stage('Maven Compile') 
            // {
            //     steps {
            //         sh 'mvn compile'
            //     }
            // }
        // stage('Maven Test') 
        // {
        //     steps {
        //         sh 'mvn test'
        //     }
        // }
        // stage('Maven Install') 
        // {
        //     steps {
        //         sh 'mvn install'
        //     }
        // }
        // stage(' Trivy Scan')
        // {
        //     steps {
        //         echo "Trivy Scan Started"
        //         sh 'trivy fs --format table --output trivy-report.txt --severity HIGH,CRITICAL .'
        //         echo "Trivy Scan Finished"
        //     }
        // }

        // stage('Sonar Analysis')
        // {
        //     environment {
        //         SCANNER_HOME = tool 'Sonar-scanner'
        //     }
        //   steps {
        //       withSonarQubeEnv('sonarserver') {
        //         sh '''${SCANNER_HOME}/bin/sonar-scanner \
        //         -Dsonar.organization=bkrrajmali \
        //         -Dsonar.projectName=springbootapp \
        //         -Dsonar.projectKey=springbootapp \
        //         -Dsonar.java.binaries=.
        //         '''
        //       }
        //     }
        // }
        stage('Maven Package') 
        {
            steps {
                sh 'mvn package'
            }
        }
    //     stage('Sonar Quality Gate') 
    //     {
    //         steps {
    //             timeout(time: 1, unit: 'MINUTES') {
    //                 waitForQuality abortPipeline: true, credentialsId: 'sonar'
    //                 echo "Sonar Quality Gate Finished"
    //         }
    //     }
    //   }
      stage ('Docker Build')
      {
        steps {
            script {
                echo "Build Docker Image"
                docker.build  ("${IMAGE_NAME}:${IMAGE_TAG}")
      
        }
      }
   }
    
   stage ('Docker Push')
   {
    steps 
    {
        script {
            echo"Docker Image Push"
            sh '''
            docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE_NAME}
            docker push ${FULL_IMAGE_NAME}
            '''
        }
    }
}
stage('Azure Login and AKS Deployment'){
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'azure-acr-spn',
            usernameVariable: 'AZURE_USERNAME',
            passwordVariable: 'AZURE_PASSWORD'
        )]) {
            script {
                echo "Azure Login Test"
                sh '''
                az login --service-principal \
                -u $AZURE_USERNAME \
                -p $AZURE_PASSWORD \
                --tenant $TENANT_ID

                az account set --subscription "a8b34663-c89f-4392-807c-03c49e0ced6b"

                az acr login --name $ACR_NAME

                az aks get-credentials \
                --resource-group $RG \
                --name $NAME \
                --overwrite-existing

                kubectl apply -f k8s/sprinboot-deployment.yaml
                '''
            }
        }
    }

}


}   // closes stages

}   // closes pipeline



