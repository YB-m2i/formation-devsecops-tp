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
          sh "mvn test"
        }
      }
//--------------------------------------------------------------------------
      stage('Mutation Tests - PIT') {
        steps {
          sh "mvn org.pitest:pitest-maven:mutationCoverage"
        }
      }
//--------------------------
}
}

// Compris