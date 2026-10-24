pipeline {
    agent any

    environment {
        AWS_REGION = 'us-west-2'
        AWS_ACCOUNT_ID = '467506934869'
        ECR_REPOSITORY = 'cloudforge'
        AWS_ROLE_CREDENTIALS = 'cloudforge-aws-role-creds'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out CloudForge source code'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -eu
                    docker build -t cloudforge:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Authenticate with AWS') {
            steps {
                withCredentials([
                    file(
                        credentialsId: "${AWS_ROLE_CREDENTIALS}",
                        variable: 'AWS_CREDS_FILE'
                    )
                ]) {
                    sh '''
                        set -eu
                        set +x

                        eval "$(python3 -c '
import json, shlex
with open("'"$AWS_CREDS_FILE"'") as f:
    c = json.load(f)
print("export AWS_ACCESS_KEY_ID=" + shlex.quote(c["AccessKeyId"]))
print("export AWS_SECRET_ACCESS_KEY=" + shlex.quote(c["SecretAccessKey"]))
print("export AWS_SESSION_TOKEN=" + shlex.quote(c["SessionToken"]))
')"

                        export AWS_DEFAULT_REGION="$AWS_REGION"

                        aws sts get-caller-identity \
                            --query Arn \
                            --output text \
                            --no-cli-pager
                    '''
                }
            }
        }

        stage('Push Image to Amazon ECR') {
            steps {
                withCredentials([
                    file(
                        credentialsId: "${AWS_ROLE_CREDENTIALS}",
                        variable: 'AWS_CREDS_FILE'
                    )
                ]) {
                    sh '''
                        set -eu
                        set +x

                        eval "$(python3 -c '
import json, shlex
with open("'"$AWS_CREDS_FILE"'") as f:
    c = json.load(f)
print("export AWS_ACCESS_KEY_ID=" + shlex.quote(c["AccessKeyId"]))
print("export AWS_SECRET_ACCESS_KEY=" + shlex.quote(c["SecretAccessKey"]))
print("export AWS_SESSION_TOKEN=" + shlex.quote(c["SessionToken"]))
')"

                        export AWS_DEFAULT_REGION="$AWS_REGION"

                        REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                        IMAGE="${REGISTRY}/${ECR_REPOSITORY}"

                        aws ecr get-login-password \
                            --region "$AWS_REGION" \
                        | docker login \
                            --username AWS \
                            --password-stdin "$REGISTRY"

                        docker tag cloudforge:${BUILD_NUMBER} "$IMAGE:${BUILD_NUMBER}"
                        docker tag cloudforge:${BUILD_NUMBER} "$IMAGE:latest"

                        docker push "$IMAGE:${BUILD_NUMBER}"
                        docker push "$IMAGE:latest"

                        docker logout "$REGISTRY"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CloudForge image built and pushed to Amazon ECR successfully!'
        }

        failure {
            echo 'CloudForge pipeline failed. Check the Jenkins console output.'
        }

        always {
            echo 'CloudForge CI/CD pipeline finished.'
        }
    }
}