pipeline {
    agent any
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    tools {
        maven 'maven3'
        jdk 'jdk-17'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                script {
                    deleteDir()  // This clears the workspace before proceeding
                }
            }
        }

        stage('git checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/ygminds73/Ekart.git'
            }
        }

        stage('Replace pom.xml') {
            steps {
                script {
                    // Download the pom.xml from the GitHub URL using curl
                    sh 'curl -L -o pom.xml https://raw.githubusercontent.com/akash08-ak/Ekart/master/pom.xml'
                }
            }
        }

        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('unit tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "${env.SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=EKART \
                        -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=target/classes"
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }

        stage('deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk-17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }

        stage('build and Tag docker image') {
            steps {
                script {
                    sh "docker build -t akashshewale0801/ekart:latest -f docker/Dockerfile ."
                }
            }
        }

        stage('Push image to Hub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'dockerhubpwd')]) {
                        sh 'docker login -u akashshewale0801 -p ${dockerhubpwd}'
                    }
                    sh 'docker push akashshewale0801/ekart:latest'
                }
            }
        }

        stage('EKS and Kubectl configuration') {
            steps {
                script {
                    sh 'aws eks update-kubeconfig --region ap-south-1 --name ankit-cluster'
                }
            }
        }

        stage('Deploy to k8s') {
            steps {
                script {
                    sh 'kubectl apply -f deploymentservice.yml'
                }
            }
        }
    }
}
