pipeline {
    agent any

    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply?')
        choice(name: 'action', choices: ['apply', 'destroy'], description: 'Select action')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_DEFAULT_REGION    = 'us-east-2'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Sonalkawale9/terraform-jenkins-pipeline.git'
            }
        }
        stage('Terraform Init') {
            steps {
                sh 'terraform init -input=false'
            }
        }
        stage('Terraform Plan') {
            when { expression { params.action == 'apply' } }
            steps {
                sh 'terraform plan -out=tfplan -input=false'
                sh 'terraform show -no-color tfplan > tfplan.txt'
            }
        }
        stage('Approval') {
            // Only ask for approval if autoApprove is false AND we are doing an apply
            when {
                allOf {
                    expression { params.autoApprove == false }
                    expression { params.action == 'apply' }
                }
            }
            steps {
                script {
                    def plan = readFile 'tfplan.txt'
                    input message: "Review Plan", parameters: [text(name: 'PlanOutput', defaultValue: plan)]
                }
            }
        }
        stage('Terraform Execution') {
            steps {
                script {
                    if (params.action == 'apply') {
                        // Applying a plan file never asks for 'yes'
                        sh "terraform apply -input=false tfplan"
                    } else {
                        sh "terraform destroy -input=false --auto-approve"
                    }
                }
            }
        }
    }
}