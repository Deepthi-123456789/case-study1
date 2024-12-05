pipeline {
    agent any

    parameters {
        choice(name: 'action', choices: ['create', 'delete'], description: 'Choose create/Destroy')
        string(name: 'ImageName', description: "name of the docker build", defaultValue: 'web')
        string(name: 'ImageTag', description: "tag of the docker build", defaultValue: 'v1')
        string(name: 'DockerHubUser', description: "name of the Application", defaultValue: 'deepthi555')
    }
    
    environment { 
        packageVersion = '1.0.1'
        nexusURL = '3.80.145.144:8081'
    }

    stages {
        stage('Checkout SCM') 
        {
            when { expression { params.action == 'create' } }
            steps {
                sh 'rm -rf case-study1'
                sh 'git clone https://github.com/Deepthi-123456789/case-study1.git'
            }
        }

        stage('Build') {
            steps {
                sh """
                    ls -la
                    zip -q -r web.zip ./* -x ".git" -x "*.zip"
                    ls -ltr
                """
            }
        }

        stage('Publish Artifact') {
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

        stage('Docker Image Build') 
        {
            when { expression { params.action == 'create' } }
            steps {
                echo "Starting Docker Image Build Stage"
                script {
                    sh "docker build -t ${params.DockerHubUser}/${params.ImageName}:${params.ImageTag} ."
                }
                echo "Docker Image Build completed"
            }
        }

        stage('Docker Image Push : DockerHub') 
        {
            when { expression { params.action == 'create' } }
            steps {
                echo "Starting Docker Image Push Stage"
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) 
                    {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        sh "docker push ${params.DockerHubUser}/${params.ImageName}:${params.ImageTag}"
                    }
                }
                echo "Docker Image Push completed"
            }
        }
        stage('Deploy to Kubernetes') 
        {
            when { expression { params.action == 'create' } }
            steps {
                echo "Starting Deploy to Kubernetes Stage"
                script {
                    try {
                        // Configure AWS CLI with credentials
                        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) 
                        {   
                            sh '''
                                if ! aws sts get-caller-identity > /dev/null; then
                                    echo "Configuring AWS CLI..."
                                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                                    aws configure set region us-east-1  # Adjust region if needed
                                else
                                    echo "AWS CLI already configured."
                                fi
                            '''
                        }

                        // Helm deployment command
                        sh """
                        cd web
                        helm upgrade --install web . --namespace default
                        """
                    } 
                    catch (Exception e) 
                    {
                        echo "Helm deployment failed."
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
                echo "Deploy to Kubernetes completed"
            }
        }
    }  

    post {
        always {
            // Clean up or any post-action after the pipeline execution
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
