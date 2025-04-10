pipeline {
  agent any
 
  stages {

      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   
//--------------------------------------------------------------------------
      stage('test unitaire') {
        steps {
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
              sh "mvn test"
            }
        }
      }
//--------------------------------------------------------------------------
      stage('Mutation Tests - PIT') {
        steps {
          sh "mvn org.pitest:pitest-maven:mutationCoverage"
        }
      }
//--------------------------
      stage('Docker Build and Push') {
        steps {
          withCredentials([string(credentialsId: 'DOCKER_HUB_PASSWORD_Formation13', variable: 'password')]) {
            sh 'sudo docker login -u ybm2i -p $password'
            sh 'printenv'
            sh 'sudo docker build -t ybm2i/devops-app:""$GIT_COMMIT"" .'
            sh 'sudo docker push ybm2i/devops-app:""$GIT_COMMIT""'
          }
 
        }
      }
//-----------------------------------------------------------------------------------------------------------
      stage('Deployment Kubernetes  ') {
        steps {
          withKubeConfig([credentialsId: 'kubeconfigyannb']) {
            sh "sed -i 's#replace#ybm2i/devops-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
            sh "kubectl apply -f k8s_deployment_service.yaml"
          }
        }
 
    }
}
}

// Compris