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
        }

	stage('Mutation Tests - PIT') {
             steps {
               sh "mvn org.pitest:pitest-maven:mutationCoverage"
             }
        }

	stage('SonarQube - SAST') {
             steps {
	         withSonarQubeEnv('SonarQube') {
                   sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops-tgdemo.eastus.cloudapp.azure.com:9000 -Dsonar.login=2078acc91cf61522ea4d6c7dae1814fc4e52d0dd"
                 }
		 timeout(time: 2, unit: 'MINUTES') {
		   script {
		     waitForQualityGate abortPipeline: true
		   }
		 }
	     }
        }

	stage('Vulnerability Scan - Docker ') {
          steps {
	    parallel(
              "Dependency Scan": {
                sh "mvn dependency-check:check"
              },
              "Trivy Scan": {
                sh "bash trivy-docker-image-scan.sh"
              }
            )
          }
        }

	stage('Docker Build and Push') {
             steps {
               withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
               sh 'printenv'
               sh 'sudo docker build -t jbadru/numeric-app:""$GIT_COMMIT"" .'
               sh 'sudo docker push jbadru/numeric-app:""$GIT_COMMIT""'
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
      post {
           always {
              junit 'target/surefire-reports/*.xml'
              jacoco execPattern: 'target/jacoco.exec'
              pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
              dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
           }
      }
}
