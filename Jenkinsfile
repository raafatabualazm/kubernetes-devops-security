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
