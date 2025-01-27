pipeline{
    agent any
    stages{
        stage("Build Artifact - Maven"){
            steps{
                sh "mvn clean package -DskipTests=true"
                archive 'target/*.jar'
            }
        }
        stage("Unit Testing - Maven"){
            steps{
                sh "mvn test"
            }

            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco execPattern: 'target/jacoco.exec'
                }
            }
         }

         stage("Perform Mutuation Testing"){
            steps{
                sh '''
                    mvn org.pitest:pitest-maven:mutationCoverage
                '''
            }

            post {
                always {
                    junit '**/target/pit-reports/**/mutations.xml'
                }
            }
         }

         stage("Code Quality Analysis"){
            
            steps{
                withSonarQubeEnv('sonarqube') {
                sh '''
                    mvn sonar:sonar -Dsonar.projectKey=devsecops-numeric-application -Dsonar.host.url=https://30012-port-imzqtskcwprqknew.labs.kodekloud.com/
                '''
            }
                timeout(time: 2, unit: 'MINUTES') {
                    script {
                        
                        waitForQualityGate abortPipeline: true
                        
                    }
                    
                }
            
            }

         }

         stage("Maven Dependency Check"){
            steps{
                sh "mvn org.owasp:dependency-check-maven:check"
            }
            post {
                always {
                    dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
                }
            }
         }  

         stage("Docker image scan with Trivy"){
            steps{
                sh '''
                    trivy --exit-code 0 --severity HIGH --no-progress openjdk:8-jdk-alpine
                    trivy --exit-code 1 --severity CRITICAL --no-progress openjdk:8-jdk-alpine
                '''
            }
         }

         stage("Conftest scan Dockerfile"){
            steps{
                sh '''
                    conftest test --policy opa-docker-security.rego Dockerfile
                '''
            }
         }

         stage("Conftest scan Kubernetes"){
            steps{
                sh '''
                    conftest test --policy opa-k8s-security.rego Dockerfile
                '''
            }
         }

         stage("Push to Docker") {
            steps {
                sh 'docker build -t docker-registry:5000/java-app:latest .'
                sh 'docker push docker-registry:5000/java-app:latest'
            }
         }

         stage("Deploy to Kubernetes") {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig', serverUrl: 'https://kubernetes.default.svc.cluster.local']) {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                }
            }
         }   
    }
}