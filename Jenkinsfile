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
        nexusURL = '3.80.145.144:8081'
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

        stage('Docker Image Build') 
        {
            when { expression { params.action == 'create' } }
            steps 
            {
                sh "docker build -t ${params.DockerHubUser}/${params.ImageName}:${params.ImageTag} ."
            }
        }

        stage('Docker Image Push : DockerHub') 
        {
            when { expression { params.action == 'create' } }
            steps 
            {
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) 
                {
                    sh '''
                        #!/bin/bash
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        docker push ${params.DockerHubUser}/${params.ImageName}:${params.ImageTag}
                    '''
                }
            }
        }

        stage('Deploy to Minikube') 
        {
            when { expression { params.action == 'create' } }
            steps 
            {
                sh '''
                    if ! minikube status > /dev/null; then
                        minikube start
                    fi
                '''
                sh '''
                    kubectl config use-context minikube
                '''
                sh '''
                    cd web
                    helm upgrade --install web . --namespace default --create-namespace
                '''
            }
        }
    }

    post 
    {
        always 
        {
            echo "Pipeline execution complete!"
        }
        success 
        {
            echo "CI/CD Pipeline succeeded!"
        }
        failure 
        {
            echo "CI/CD Pipeline failed."
        }
    }
}
