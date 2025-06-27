pipeline {
    agent any
    environment {
        AWS_REGION = 'ap-south-1'
        S3_BUCKET = 's3bucket-moso-interior'
        CF_DIST_ID = 'EN3AB08UJKYZ3'
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm  // Simplified checkout using the SCM that triggered the build
            }
        }
        stage('Deploy to S3 & CloudFront') {
            steps {
                withCredentials([
                    string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                        # Configure AWS CLI
                        aws configure set aws_access_key_id "$AWS_ACCESS_KEY_ID"
                        aws configure set aws_secret_access_key "$AWS_SECRET_ACCESS_KEY"
                        aws configure set region "$AWS_REGION"

                        # Clean up unwanted files
                        find . -name "*:Zone.Identifier" -delete
                        rm -f .*.swp || true

                        # Sync to S3
                        aws s3 sync . s3://$S3_BUCKET/ \\
                            --exclude ".git/*" \\
                            --exclude "Jenkinsfile" \\
                            --delete

                        # Invalidate CloudFront cache
                        aws cloudfront create-invalidation \\
                            --distribution-id $CF_DIST_ID \\
                            --paths "/*"
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'Deployment process completed'
        }
    }
}
