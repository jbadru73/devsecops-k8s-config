pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
	      echo "building Artifact and Archiving jar file"
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   
    }
}
