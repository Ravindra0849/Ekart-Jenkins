pipeline {
        agent any
        tools{
            jdk  'jdk17'
            maven  'maven3'
        }
        
        environment {
            SCANNER_HOME=tool 'sonar-scanner'
        }
    
        stages {

            stage('Clean Workspace') {
                steps {
                    cleanWs()
                }
            }

            stage('Git Checkout') {
                steps {
                    git 'https://github.com/Ravindra0849/Ekart-Jenkins.git'
                }
            }
            
            stage('COMPILE') {
                steps {
                    sh "mvn clean compile -DskipTests=true"
                }
            }
            
            stage('OWASP FS SCAN') {
                steps {
                    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
            
            stage('TRIVY FS SCAN') {
                steps {
                    sh "trivy fs . > trivyfs.txt"
                }
            }
            
            stage('Sonarqube') {
                steps {
                    withSonarQubeEnv('sonar'){
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Ekart \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=Ekart '''
                    }
                }
            }
            
            stage('Build') {
                steps {
                    sh "mvn clean package -DskipTests=true"
                }
            }
            
            stage('Deploy to Nexus') {
                steps {
                    withMaven(globalMavenSettingsConfig: 'global-settings-xml') {
                        sh "mvn deploy -DskipTests=true"
                    }
                }
            }
            
            stage("Docker Build & Tag"){
                steps{
                    script{
                        withDockerRegistry(credentialsId: 'Dockerhub', toolName: 'docker'){   
                            sh "docker build -t ekart -f docker/Dockerfile ."
                            sh "docker tag ekart ravisree900/ekart:'${env.BUILD_NUMBER}'"
                        }
                    }
                }
            }
            
            stage("Trivy Image Scan"){
                steps{
                    sh "trivy image ravisree900/ekart:'${env.BUILD_NUMBER}' > trivyimage.txt" 
                }
            }
            
            stage("Docker Push"){
                steps{
                    script{
                        withDockerRegistry(credentialsId: 'Dockerhub', toolName: 'docker'){   
                            sh "docker push ravisree900/ekart:'${env.BUILD_NUMBER}'"
                        }
                    }
                }
            }
            
            stage("Deploy to Container"){
                steps{
                    script{
                        withDockerRegistry(credentialsId: 'Dockerhub', toolName: 'docker'){   
                            sh "docker run -d --name ekart -p 8070:8070 ravisree900/ekart:'${env.BUILD_NUMBER}'"
                        }
                    }
                }
            }
        }
    }