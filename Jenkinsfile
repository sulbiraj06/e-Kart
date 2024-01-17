pipeline{
    agent any
        tools {
            maven 'maven3'
            jdk 'jdk17'
        }
        environment {
            SCANNER_HOME= tool 'sonar-scanner'
        }
    stages {
        stage("Git Checkout"){
            steps{
                git branch: 'master', url: 'https://github.com/sulbiraj06/e-Kart.git'
            }
        }
        stage("Compile"){
            steps{
                sh "mvn compile"
            }
        }
        stage("Unit Tests"){
            steps{
                sh "mvn test -DskipTests=true"
            }
        }
        stage("SonarQube Analysis"){
            steps{
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                    -Dsonar.java.binaries=. '''

                }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('Deploy To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage('Docker Build & tag Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName:'docker') {
                        sh "docker build -t sulbiraj/ekart:latest -f docker/Dockerfile ."
                    }
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                script {
                    sh "trivy image sulbiraj/ekart:latest > trivy-report.txt "
                }
            }
        }
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName:'docker') {
                         sh "docker push sulbiraj/ekart:latest"
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://51372781A9E2E74A756C425DB9C66630.gr7.us-east-1.eks.amazonaws.com') {
                        sh "kubectl apply -f deploymentservice.yml -n webapps" 
                        sh "kubectl get svc -n webapps"
                    }
            }
        }
    }
}
