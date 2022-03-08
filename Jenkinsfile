pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
	      echo "Building Artifact and Archiving jar file"
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }

       stage('Unit Tests') {
            steps {
              echo "Testing Application"
              sh "mvn test"
	    }
	    post {
	      always {
	        junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
	      }
            }
        }

	stage('Docker Build and Push') {
             steps {
               withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
               sh 'printenv'
               sh 'docker build -t jbadru/numeric-app:""$GIT_COMMIT"" .'
               sh 'docker push jbadru/numeric-app:""$GIT_COMMIT""'
               }
             }  
        }
    }
}
