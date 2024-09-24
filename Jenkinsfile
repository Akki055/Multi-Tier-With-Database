pipeline {
    agent any
    
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Akki055/Multi-Tier-With-Database.git'
            }
        }
        
        stage('Compile Code') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test Code') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        
        stage('Trivy Code Scan') {
            steps {
                sh 'trivy fs --format table -o fs.html .'
            }
        }
        
        stage('Sonar Scanning') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=bankapp -Dsonar.projectName=bankapp  \
                    -Dsonar.java.binaries=target '''
                }
            }
        }

        stage('Building_App') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }
        
        stage('OWASP Dependency-Check Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan target/', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Deploy_Artifact_to_Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-config', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }
        
        stage('Docker_Image_Build_and_Tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'doc-cred') {
                        sh 'docker build -t akki058/bankapp:latest .'
                    }
                }
            }
        }
        
        stage('Trivy_Image_scan') {
            steps {
                sh 'trivy image --format table -o image.html akki058/bankapp:latest'
            }
        }
        
        stage('Docker_Image_Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'doc-cred') {
                        sh 'docker push akki058/bankapp:latest'
                    }
                }
            }
        }
        
        stage('K8s_deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'local', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: ' https://103.127.30.209:6443') {
                    sh 'kubectl apply -f deployment-service.yml'
                }
            }
        }
        
        stage('K8s_Check_Services') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'local', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: ' https://103.127.30.209:6443') {
                    sh 'kubectl get pods -n webapps'
                    sh 'kubectl get svc -n webapps'
                }
            }
        }
    }
}

