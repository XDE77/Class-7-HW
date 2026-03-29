pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        TF_IN_AUTOMATION   = 'true'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'Buneary-Jenk']]) {
  sh '''
    echo "Caller identity:"
    aws sts get-caller-identity --output json

    echo "Bucket location:"
    aws s3api get-bucket-location --bucket terraform-state-aaronmcd || true

    echo "Try head-object (will show error code if forbidden):"
    aws s3api head-object --bucket terraform-state-aaronmcd --key jenkins-test-031726.tfstate || true

    echo "List bucket (may be restricted):"
    aws s3 ls s3://terraform-state-aaronmcd/ || true
  '''
}


        stage('Terraform Init') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'Buneary-Jenk'
                ]]) {
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'Buneary-Jenk'
                ]]) {
                    sh '''
                        terraform plan -out=tfplan
                        terraform apply -auto-approve tfplan
                    '''
                }
            }
        }

        stage('Optional Destroy') {
            steps {
                script {
                    def destroyChoice = input(
                        message: 'Do you want to run terraform destroy?',
                        ok: 'Submit',
                        parameters: [
                            choice(
                                name: 'DESTROY',
                                choices: ['no', 'yes'],
                                description: 'Select yes to destroy resources'
                            )
                        ]
                    )

                    if (destroyChoice == 'yes') {
                        withCredentials([[
                            $class: 'AmazonWebServicesCredentialsBinding',
                            credentialsId: 'Buneary-Jenk'
                        ]]) {
                            sh 'terraform destroy -auto-approve'
                        }
                    } else {
                        echo "Skipping destroy"
                    }
                }
            }
        }
    }
}
