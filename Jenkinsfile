pipeline {
    agent any

    environment {
        DOCKERHUB_REPO        = 'manasi99/book-liabrary'   // change to your Docker Hub repo
        IMAGE_TAG             = "${env.BUILD_NUMBER}"
        AWS_REGION            = 'us-east-1'
        ECS_CLUSTER           = 'jenkins-ecs-cluster'           // must match ecs_cluster_name in ecs.tf
        ECS_SERVICE           = 'app-service'                   // must match ecs_service_name in ecs.tf
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')  // Jenkins credential ID (username/password)
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} -t ${DOCKERHUB_REPO}:latest ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh '''
                    echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "$DOCKERHUB_CREDENTIALS_USR" --password-stdin
                    docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}
                    docker push ${DOCKERHUB_REPO}:latest
                '''
            }
        }

        stage('Deploy to ECS') {
            steps {
                // Uses the AWS Credentials Binding plugin; credential ID 'aws-creds'
                // must be set up in Jenkins (Manage Jenkins > Credentials) with an
                // AWS access key / secret key pair.
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh '''
                        aws ecs update-service \
                          --cluster ${ECS_CLUSTER} \
                          --service ${ECS_SERVICE} \
                          --force-new-deployment \
                          --region ${AWS_REGION}
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        success {
            echo "Pushed ${DOCKERHUB_REPO}:${IMAGE_TAG} and triggered ECS deployment on ${ECS_CLUSTER}/${ECS_SERVICE}."
        }
        failure {
            echo "Pipeline failed — check the stage logs above."
        }
    }
}
