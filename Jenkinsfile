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

	stage('Mutation Tests - PIT') {
             steps {
               sh "mvn org.pitest:pitest-maven:mutationCoverage"
             }
             post {
               always {
                 pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
               }
             }
        }

	stage('SonarQube - SAST') {
             steps {
               sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops-demo.eastus.cloudapp.azure.com:9000 -Dsonar.login=0925129cf435c63164d3e63c9f9d88ea9f9d7f05"
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

	stage('Kubernetes Deployment - DEV') {
             steps {
               withKubeConfig([credentialsId: 'kubeconfig']) {
               sh "sed -i 's#replace#jbadru/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
               sh "kubectl apply -f k8s_deployment_service.yaml"
               }
             }
         }
    }
}
