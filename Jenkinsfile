pipeline {
    agent any
    environment {
        AWS_REGION = 'ap-south-1'
        S3_BUCKET = 's3bucket-moso-interior'
        CF_DIST_ID = 'EN3AB08UJKYZ3'
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        url: 'https://github.com/PrabhaAjit/my-static-website.git',
                        credentialsId: '' // Add credential ID if private repo
                    ]]
                ])
            }
        }
        stage('Deploy to S3 & CloudFront') {
            steps {
                script {
                    sh '''
                        # Clean up unwanted files
                        find . -name "*:Zone.Identifier" -delete
                        rm -f .*.swp || true

                        # Sync to S3
                        aws s3 sync . s3://$S3_BUCKET/ \\
                            --exclude ".git/*" \\
                            --exclude "Jenkinsfile" \\
                            --delete \\
                            --region $AWS_REGION

                        # Invalidate CloudFront cache
                        aws cloudfront create-invalidation \\
                            --distribution-id $CF_DIST_ID \\
                            --paths "/*" \\
                            --region $AWS_REGION
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'Deployment process completed'
            // cleanWs() // Uncomment to clean workspace
        }
    }
}
