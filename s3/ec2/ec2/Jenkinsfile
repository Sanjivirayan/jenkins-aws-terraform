
pipeline {
    agent any

    parameters {
        choice(
            name: 'RESOURCE',
            choices: ['s3', 'ec2'],
            description: 'Choose AWS resource to create'
        )
    }

    environment {
        AWS_DEFAULT_REGION = 'ap-southeast-2'
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    stages {

        stage('Check Tools') {
            steps {
                sh '''
                    terraform version
                    aws sts get-caller-identity
                '''
            }
        }

        stage('GitHub Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                dir("${params.RESOURCE}") {
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                dir("${params.RESOURCE}") {
                    sh 'terraform validate'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir("${params.RESOURCE}") {
                    sh 'terraform plan -out=tfplan'
                }
            }
        }

        stage('Approval') {
            steps {
                input message: "Approve creation of AWS ${params.RESOURCE} resource?",
                      ok: 'Create Resource'
            }
        }

        stage('Terraform Apply') {
            steps {
                dir("${params.RESOURCE}") {
                    sh 'terraform apply -auto-approve tfplan'
                }
            }
        }
    }

    post {
        success {
            echo 'AWS resource created successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins console output.'
        }
    }
}

