pipeline {
    agent any

    environment {
        // Variáveis estáticas que podem ser definidas aqui ou dinamicamente dentro dos stages
        STACK_NAME = 'nome-da-sua-stack'
        TEMPLATE_FILE = 'caminho/para/s3-bucket-template.yaml'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Load Config') {
            steps {
                script {
                    // Carrega o arquivo YAML
                    def config = readYaml file: 'config.yaml'

                    // Define as variáveis de ambiente com base no branch
                    if (env.BRANCH_NAME == 'dev' || env.BRANCH_NAME.startsWith('feature/')) {
                        env.AWS_ACCOUNT = config.deploy.aws.dev.account
                        env.AWS_REGION = config.deploy.aws.dev.region
                        env.PARAMS_FILE = 'caminho/para/dev-params.json'
                    } else {
                        env.AWS_ACCOUNT = config.deploy.aws.prod.account
                        env.AWS_REGION = config.deploy.aws.prod.region
                        env.PARAMS_FILE = 'caminho/para/prod-params.json'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Usa as variáveis de ambiente configuradas anteriormente
                    sh "aws cloudformation deploy --template-file ${env.TEMPLATE_FILE} --stack-name ${env.STACK_NAME} --parameter-overrides file://${env.PARAMS_FILE} --region ${env.AWS_REGION} --capabilities CAPABILITY_IAM"
                }
            }
        }
    }

    post {
        success {
            echo 'Deploy realizado com sucesso em região ${env.AWS_REGION} para a conta ${env.AWS_ACCOUNT}.'
        }
        failure {
            echo 'Falha no deploy.'
        }
    }
}
