pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   
      stage('Unit Testig') {
            steps {
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
        }
      stage('SonarQube - SAST') {
            steps {
              withSonarQubeEnv('sonarqube') {
                sh "mvn sonar:sonar \
                        -Dsonar.projectKey=numeric-application \
                        -Dsonar.host.url=http://devsecops-ravi-demo.eastus.cloudapp.azure.com:9000/"
              }
              timeout(time: 2, unit: 'MINUTES') {
                script {
                  waitForQualityGate abortPipeline: true
                }
              }
            }
        }

      stage('Docker Build and Push') {
            steps {
              withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                sh 'printenv'
                sh 'sudo docker build -t ravicsa12/numeric-app:""$GIT_COMMIT"" .'
                sh 'docker push ravicsa12/numeric-app:""$GIT_COMMIT""'
             }
           }
        }
      stage('K8S Deployment - DEV') {
            steps {
               withKubeConfig([credentialsId: 'kubeconfig']) {
                  sh "sed -i 's#replace#ravicsa12/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                  sh "kubectl apply -f k8s_deployment_service.yaml"
            }
          }
       }
    }
}
