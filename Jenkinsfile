pipeline {
    agent any
    environment {
        PROJECT_ID = 'beaming-force-358817'
        CLUSTER_NAME = 'gke'
        LOCATION = 'us-central1'
        CREDENTIALS_ID = 'gke'
    }
    stages {
        stage("Checkout code") {
            steps {
                checkout scm
            }
        }
        stage("Build image") {
            steps {
                script {
                    myapp = docker.build("nixonlauture/hello:${env.BUILD_ID}")
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
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                            myapp.push("latest")
                            myapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }
        stage("Pushing ECR Image to GCR") {
      steps {
        script {
          withDockerRegistry([CREDENTIALS_ID: "gcr:${params.GCP_PROJECT_ID}", url: "https://gcr.io"]) {
            sh "docker push gcr.io/${params.GCP_PROJECT_ID}/${params.GCR_IMAGE_NAME}:${params.GCR_IMAGE_TAG}"
          }
        }
      }

        stage('Deploy to GKE') {
            steps{
                sh "sed -i 's/hello:latest/hello:${env.BUILD_ID}/g' deployment.yaml"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            }
        }
    }
}
