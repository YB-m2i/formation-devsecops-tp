pipeline {
  agent any
 
  stages {

      stage('Build Artifact') {
            steps {
              sh "sudo mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   
//-----------------------------------------------------------------------------
              stage('test unitaire ') {
                    steps {
                      catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                      sh "sudo mvn test"
                      }
                    
                    }
                post{
                  always{
                    junit 'target/surefire-reports/*.xml'
                  
                  }
                }
                }
                //--------------------------
            stage('Mutation Tests - PIT') {
              steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                sh "sudo mvn org.pitest:pitest-maven:mutationCoverage"
                }
              }
              post{
                always{
                pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
              
                }
              }
        
            }
//-------------------------------------------------------------------------------------------
       stage('scan sonarqube') {
        steps {
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
              sh "sudo mvn clean verify sonar:sonar \
              -Dsonar.projectKey=TP-YB \
              -Dsonar.projectName='TP-YB' \
              -Dsonar.host.url=http://devopstssr.eastus.cloudapp.azure.com:9998 \
              -Dsonar.token=sqp_87f06e7844e609d5becc436d7afedff782873289"
            }
        }
            
        
      }
        /////////////////////////
 
    stage('Vulnerability Scan - Docker') {
          steps {
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
              sh "sudo mvn dependency-check:check"
            }
          }
          post {
            always {
              dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
              jacoco(execPattern: 'target/jacoco.exec')
            }
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

// Compris bis