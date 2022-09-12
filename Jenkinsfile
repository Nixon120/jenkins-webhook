pipeline {
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
        stage('Build Container Image') {
         
            steps {
                unstash 'dist'
                sh "docker build . -t gcr.io/beaming-force-358817/gke-gcr:hello"
                sh "gcloud docker -- push gcr.io/beaming-force-358817/gke-gcr:hello"
                sh "gcloud container images add-tag gcr.io/beaming-force-358817/gke-gcr:hello gcr.io/beaming-force-358817/gke-gcr:hello"
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
