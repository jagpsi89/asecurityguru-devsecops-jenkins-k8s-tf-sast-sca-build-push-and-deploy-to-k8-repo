pipeline {
    agent any

    tools {
        maven 'Maven_3_5_2'
    }

    stages {
        stage('Compile and Run Sonar Analysis') {
            steps {
                echo '=== Compiling and Running Sonar Analysis ==='
                sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=jaggibuggywebapp -Dsonar.organization=jaggibuggywebapp -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=537ef5d69358afdb3de284581df576177fbabfb5'
            }
        }

        stage('Run SCA Analysis Using Snyk') {
            steps {
                echo '=== Running SCA Analysis Using Snyk ==='
                withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                    sh 'mvn snyk:test -fn'
                }
            }
        }

        stage('Build') {
            steps {
                echo '=== Building Docker Image ==='
                withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                    script {
                        app = docker.build("jaggiimages")
                    }
                }
            }
        }

        stage('Push') {
            steps {
                echo '=== Pushing Docker Image ==='
                script {
                    docker.withRegistry('https://339656718719.dkr.ecr.us-west-2.amazonaws.com', 'ecr:us-west-2:aws-credentials') {
                        app.push("latest")
                    }
                }
            }
        }

        stage('Kubernetes Deployment of ASG Bugg Web Application') {
    steps {
        script {
            echo '=== Deploying to Kubernetes ==='
            try {
                withKubeConfig([credentialsId: 'kubelogin']) {
                    sh('kubectl delete all --all -n devsecops')
                    sh('kubectl apply -f deployment.yaml --namespace=devsecops')
                }
            } catch (Exception e) {
                echo "Error deploying to Kubernetes: ${e.message}"
                currentBuild.result = 'FAILURE'
                error("Failed to deploy to Kubernetes.")
            }
        }
    }
 }
