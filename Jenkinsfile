pipeline {
    agent { label 'Test-Node' }
    tools {
        jdk 'jdk-17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME= tool 'Sonar Scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: '3984223f-e013-448c-82f4-32a0c9870fa9', poll: false, url: 'https://github.com/srikanth-mallela/Ekart.git'
            }
        }
        stage('COMPILE') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        //stage('OWASP SCAN') {
            //steps {
                //dependencyCheck additionalArguments: '--scan ./ --format XML,HTML', odcInstallation: 'DP'
                //dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
           // }
       // }
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server') {
                sh ''' /home/ec2-user/jenkins-slave/tools/hudson.plugins.sonar.SonarRunnerInstallation/Sonar_Scanner/bin/sonar-scanner \
                -Dsonar.projectKey=Ekart \
                -Dsonar.projectName=Ekart \
                -Dsonar.java.binaries=target/classes \
                -Dsonar.propertyKey=EKart '''
                }
            }
        }
        stage('BuildnPush') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '12b3c8d1-7c25-4367-901b-d876e720768d', toolName: 'docker') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh " docker tag shopping-cart mallelas571/shopping-cart:latest"
                        sh " docker push mallelas571/shopping-cart:latest"
                     }
                    }
                }
            }
            stage('Deploy') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '12b3c8d1-7c25-4367-901b-d876e720768d', toolName: 'docker') {
                        sh "docker run -d --name shop-shop -p 8070:8070 mallelas571/shopping-cart:latest"
                        
                    }
                }
            }
        }
}
}
