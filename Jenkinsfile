pipeline {
    agent any

    parameters {
        choice(name: 'action', choices: ['create', 'delete'], description: 'Choose create/Destroy')
        string(name: 'ImageName', description: "Name of the Docker build", defaultValue: 'web')
        string(name: 'ImageTag', description: "Tag of the Docker build", defaultValue: 'v1')
        string(name: 'DockerHubUser', description: "DockerHub Username", defaultValue: 'deepthi555')
    }

    environment {
        packageVersion = '1.0.1'
        nexusURL = '44.201.183.60:8081'
    }

    stages {
        stage('Checkout SCM') {
            when { expression { params.action == 'create' } }
            steps {
                sh '''
                    rm -rf case-study1
                    git clone https://github.com/Deepthi-123456789/case-study1.git
                '''
            }
        }

        stage('Build') {
            when { expression { params.action == 'create' } }
            steps {
                sh '''
                    ls -la
                    zip -q -r web.zip ./* -x ".git" -x "*.zip"
                    ls -ltr
                '''
            }
        }

        stage('Publish Artifact') {
            when { expression { params.action == 'create' } }
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${nexusURL}",
                    groupId: 'com.roboshop',
                    version: "${packageVersion}",
                    repository: 'web',
                    credentialsId: 'nexus-auth',
                    artifacts: [
                        [artifactId: 'web',
                         classifier: '',
                         file: 'web.zip',
                         type: 'zip']
                    ]
                )
            }
        }

        stage('Docker Image Build') {
            when { expression { params.action == 'create' } }
            steps {
                sh "docker build -t ${params.DockerHubUser}/${params.ImageName}:${params.ImageTag} ."
            }
        }

        stage('Docker Image Push : DockerHub') {
            when { expression { params.action == 'create' } }
            steps {
                echo "Starting Docker Image Push Stage"
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        sh "docker push ${params.DockerHubUser}/${params.ImageName}:${params.ImageTag}"
                    }
                }
                echo "Docker Image Push completed"
            }
        }

        stage('Deploy to Kubernetes Using Helm') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    // Set kubectl context to the correct cluster (if applicable)
                    sh '''
                        kubectl config use-context <your-k8s-cluster-context>
                    '''

                    // Deploy using Helm
                    sh '''
                        cd web
                        helm upgrade --install web . --namespace default --create-namespace
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution complete!"
        }
        success {
            echo "CI/CD Pipeline succeeded!"
        }
        failure {
            echo "CI/CD Pipeline failed."
        }
    }
}
