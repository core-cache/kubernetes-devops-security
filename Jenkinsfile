pipeline {
  agent any

  stages {
      stage('Build Artifact - Maven') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }
        }
      stage('Unit Tests') {
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
                sh 'mvn org.pitest:pitest-maven:mutationCoverage'
            }

            post {
                always {
                      pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
                  }
              }

       }

       stage('SonarQube - SAST'){
            steps {
                sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://hh-devsecops-demo.eastus.cloudapp.azure.com:9000 -Dsonar.token=sqp_74e982eb4725a678d502c63269491d14e1fc7a6e"
            }

       }

       stage('Docker Build and Push') {
            steps {
                withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                    sh 'printenv'
                    sh 'docker build -t ssopen/numeric-app:""$GIT_COMMIT"" .'
                    sh 'docker push ssopen/numeric-app:""$GIT_COMMIT""'
                }
            }

       }

       stage('Kubernetes Deployment -DEV') {
            steps {
                withKubeConfig([credentialsId: "kubeconfig"]) {
                    sh "sed -i 's#replace#ssopen/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                    sh "kubectl apply -f k8s_deployment_service.yaml"
                }
            }

       }

    }
}