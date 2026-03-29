pipeline {
  agent any
  environment {
    AWS_DEFAULT_REGION = 'us-east-1'
    TF_IN_AUTOMATION   = 'true'
    TERRAFORM_DIR      = 'terraform' // change this to your .tf location
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Terraform Init') {
      steps {
        dir(env.TERRAFORM_DIR) {
          withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'Buneary-Jenk'
          ]]) {
            sh '''
              echo "Running terraform init in $(pwd)"
              terraform init -input=false
              terraform validate || true
            '''
          }
        }
      }
    }

    stage('Terraform Plan') {
      steps {
        dir(env.TERRAFORM_DIR) {
          withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'Buneary-Jenk'
          ]]) {
            sh '''
              echo "Checking AWS env presence: ${AWS_ACCESS_KEY_ID:+present}"
              terraform plan -input=false -out=tfplan
            '''
          }
        }
      }
    }

    stage('Terraform Apply') {
      when { expression { return fileExists("${env.TERRAFORM_DIR}/tfplan") } }
      steps {
        dir(env.TERRAFORM_DIR) {
          withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'Buneary-Jenk'
          ]]) {
            sh 'terraform apply -input=false -auto-approve tfplan'
          }
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
              choice(name: 'DESTROY', choices: ['no','yes'], description: 'Select yes to destroy resources')
            ]
          )
          if (destroyChoice == 'yes') {
            dir(env.TERRAFORM_DIR) {
              withCredentials([[
                $class: 'AmazonWebServicesCredentialsBinding',
                credentialsId: 'Buneary-Jenk'
              ]]) {
                sh 'terraform destroy -input=false -auto-approve'
              }
            }
          } else {
            echo "Skipping destroy"
          }
        }
      }
    }
  }
}
