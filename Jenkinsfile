pipeline {
    agent any
    tools {
        maven 'maven3'
    }
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        //stage('source code checkout') {
         //   steps {
          //      git branch: 'main', url: 'https://github.com/vuyyuru-bhanu/Multi-Tier-With-Database.git'
         //   }
       // }
        stage('compail') {
            steps {
                sh "mvn compile"
            }
        }
        stage('test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }
        stage('Sonar qube analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Multi -Dsonar.projectKey=Multi -Dsonar.java.binaries=target"
    
                    }
            }
        }
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('Push to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'settings-maven', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                        sh "mvn deploy -DskipTests=true"
                        }
            }
        }
        stage('Docker image build ') {
            steps {
             script{
                 withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker build -t bhanu3333/3-tire:latest ."
                                        }
             }
            }
        }
         stage('Trivy image Scan') {
            steps {
                sh "trivy image --format table -o fs-image-report.html bhanu3333/3-tire:latest"
            }
        }
        stage('Docker image push ') {
            steps {
             script{
                 withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker push bhanu3333/3-tire:latest"
                                        }
             }
            }
        }
        stage('Deploy to eks') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'eks-cluster', contextName: '', credentialsId: 'k8-tocken', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://e6e8bc3140e924fb22b4a4e82f577346.gr7.ap-south-1.eks.amazonaws.com') {
                       sh "kubectl apply -f ds.yml -n webapps"
                       sleep 30
                        }
            }
        }
        stage('verify the deployment ') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'eks-cluster', contextName: '', credentialsId: 'k8-tocken', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://e6e8bc3140e924fb22b4a4e82f577346.gr7.ap-south-1.eks.amazonaws.com') {
                       sh "kubectl get po -n webapps"
                       sh "kubectl get svc -n webapps"
                        }
            }
        }
        
        
        
    }
}
