pipeline {
  }
    agent any
    
    stages {
        stage("Checkout code") {
            steps {
                checkout scm
            }
        }
        stage("Build image") {
            steps {
                script {
                    myapp = docker.build("beaming-force-358817/gke-gcr:${env.BUILD_ID}")
                }
            }
        }
        
        stage ("lint dockerfile") {
            steps {
                sh '/bin/hadolint Dockerfile | tee -a hadolint_lint.txt'
            }
            post {
        always {
            archiveArtifacts 'hadolint_lint.txt'
         }
       }
    }

        stage("Push image") {
            steps {
                script {
                    docker.withRegistry('https://gcr.io', 'gcr:gke') {
                            myapp.push("latest")
                      
                    }
                }
            }
        }


        stage('Deploy to GKE') {
            steps{
                sh "sed -i 's/hello:latest/hello:${env.BUILD_ID}/g' deployment.yaml"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            }
        }
    

