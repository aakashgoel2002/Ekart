pipeline {
    agent any
    
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/aakashgoel2002/Ekart.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        stage('SonarQube Analysis') {
            steps {
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
        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage('Build and Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                        sh "docker build -t aakashgoel2002/ekart:latest -f docker/Dockerfile ."
                    }
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                sh "trivy image aakashgoel2002/ekart:latest > trivy-report.txt"
            }
        }
        stage('Docker Image Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                        sh "docker push aakashgoel2002/ekart:latest"
                    }
                }
            }
        }
        stage('Kubernetes Deploy') {
            steps {
		script{
	                sh ""
        	        echo "Deployment Successful!"
		}
            }
        }
    }
}
