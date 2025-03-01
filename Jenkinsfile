pipeline {
    agent any

    environment {
        // Set AWS credentials as environment variables to use them across all stages
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        
        // Update PATH to include the Terraform installation directory
        PATH = "C:\\Users\\Lenovo\\Downloads\\terraform_1.10.5_windows_amd64;$PATH"
    }

    stages {
        stage('Terraform init') {
            steps {
                // Execute terraform init with injected AWS credentials
                bat 'terraform init'
            }
        }

        stage('Terraform plan') {
            steps {
                // Execute terraform plan with injected AWS credentials
                bat 'terraform plan'
            }
        }

        stage('Manual Approval for Terraform Apply') {
            steps {
                script {
                    // This will pause the pipeline and wait for manual input
                    input message: 'Do you want to apply the changes?', ok: 'Proceed'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                // Execute terraform apply with injected AWS credentials
                bat 'terraform apply --auto-approve'
            }
        }

        stage('Get EC2 Public IP') {
            steps {
                script {
                    // Get the public IP of the EC2 instance created by Terraform
                    def publicIp = bat(script: 'terraform output -raw instance_public_ip', returnStdout: true).trim()
                    // Store the public IP in an environment variable
                    env.EC2_PUBLIC_IP = publicIp
                    
                    // Print the EC2 Public IP to the Jenkins console log
                    echo "EC2 Public IP: ${env.EC2_PUBLIC_IP}"
                }
            }
        }
      stage('Install HTTPD on EC2') {
         steps {
           script {            // Run the Ansible playbook to install HTTPD on the EC2 instance using the public IP
            bat 'wsl ansible-playbook -i httpd.yml'
        }
        }
      }



        stage('Manual Approval for Terraform Destroy') {
            steps {
                script {
                    // This will pause the pipeline and wait for manual input before destroying the infrastructure
                    input message: 'Do you want to destroy the infrastructure?', ok: 'Proceed'
                }
            }
        }

        stage('Terraform Destroy') {
            steps {
                // Execute terraform destroy with injected AWS credentials
                bat 'terraform destroy --auto-approve'
            }
        }
    }
}
