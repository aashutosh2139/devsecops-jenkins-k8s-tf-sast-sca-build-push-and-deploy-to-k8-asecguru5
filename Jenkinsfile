pipeline {
    agent any
    tools {
        maven 'maven__3_2_5'  // Ensure this matches your Jenkins Maven configuration
    }
    stages {
        stage('Compile and Run Sonar Analysis') {
            steps {
                sh '''
                    mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=asgbuggywebappw_asgbuggywebappw \
                    -Dsonar.organization=asgbuggywebapp \
                    -Dsonar.host.url=https://sonarcloud.io \
                    -Dsonar.token=${SONAR_TOKEN}  // Use environment variable for token
                '''
            }
        }

        stage('Run SCA Analysis Using Snyk') {
            steps {
                withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                    sh 'mvn snyk:test -fn'
                }
            }
        }

        stage('Build') {
            steps {
                withDockerRegistry([credentialsId: 'dockerlogin', url: '']) {
                    script {
                        app = docker.build('devcloudninjas')
                    }
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://221650130255.dkr.ecr.us-west-2.amazonaws.com', 'ecr:us-west-2:aws-credentials') {
                        app.push('latest')
                    }
                }
            }
        }

        stage('Kubernetes Deployment of DCN Buggy Web Application') {
            steps {
                withKubeConfig([credentialsId: 'kubelogin']) {
                    sh 'kubectl delete all --all -n devsecops'
                    sh 'kubectl apply -f deployment.yaml --namespace=devsecops'
                }
            }
        }
    }
}
