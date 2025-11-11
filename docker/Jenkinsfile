pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven1'
    }

    stages {
        stage('Checkout') {
            steps {
                git(
                    url: 'https://github.com/manish-g0u74m/Ekart-Java-Webapp.git',
                    branch: 'devops'
                )
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonar') {
                    script {
                        def scanner = tool name: 'sonar', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                        sh """
                            "${scanner}/bin/sonar-scanner" \
                            -Dsonar.projectKey="Ekart-Java-WebApp" \
                            -Dsonar.projectName="Ekart-Java-WebApp" \
                            -Dsonar.sources=src \
                            -Dsonar.java.binaries=target/classes
                        """
                    }
                }
            }
        }

        stage('Trivy file system scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }

        stage('Check SonarQube Quality Gate') {
            steps {
                echo 'Waiting for SonarQube Quality Gate result...'
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --nvdApiKey b588dcc5-7433-4160-b160-8d774e5b866', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
       stage("docker Image build") {
            steps {
                echo "Code Build Stage"
                sh "docker build -t ekart-java-webapp:latest -f ./docker/Dockerfile ."
            }
        }

        stage("TRIVY image Scane") {
            steps {
                sh "trivy image ekart-java-webapp:latest > trivy-frontend-image.txt"
           }
        }

        stage("Push To DockerHub") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "dockerHubCreds",
                    usernameVariable: "dockerHubUser",
                    passwordVariable: "dockerHubPass"
                )]) {
                    sh 'echo $dockerHubPass | docker login -u $dockerHubUser --password-stdin'
                    sh "docker image tag ekart-java-webapp:latest ${env.dockerHubUser}/ekart-java-webapp:latest"
                    
                    echo "Push Ekart-Java-Webapp image"
                    sh "docker push ${env.dockerHubUser}/ekart-java-webapp:latest"
                 }
            }
        }
        stage("Deploy on K8s") {
            steps {
                withKubeConfig([credentialsId: 'kube-cred-id']) {
                    sh "kubectl apply -f ./k8s/ekart-ns.yml"
                    sh "kubectl apply -f ./k8s/ekart-service.yml"
                    sh "kubectl apply -f ./k8s/ekart-deployment.yml"
                }
            }
        }
    }
    post {
        success {
            emailext(
                to: 'manish.sharma.devops@gmail.com',
                subject: 'Build Successful for Ekart Springboot Application',
                body: 'The build has completed successfully for Ekart Springboot Application.'
            )
        }
        failure {
            emailext(
                to: 'manish.sharma.devops@gmail.com',
                subject: 'Build Failed for Ekart Springboot Application',
                body: 'The build has failed for Ekart Springboot Application. Please check the Jenkins logs for details.'
            )
        }
    }
}
