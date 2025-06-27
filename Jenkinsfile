pipeline {
    agent any
    environment {
        AWS_REGION = 'ap-south-1'
        S3_BUCKET = 's3bucket-moso-interior'
        CF_DIST_ID = 'EN3AB08UJKYZ3'
        AWS_ACCESS_KEY = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_KEY = credentials('AWS_SECRET_ACCESS_KEY')
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
                        credentialsId: ''
                    ]]
                ])
            }
        }
        stage('Deploy to S3 & CloudFront') {
            steps {
                script {
                    sh '''
                        export AWS_ACCESS_KEY_ID="$AWS_ACCESS_KEY"
                        export AWS_SECRET_ACCESS_KEY="$AWS_SECRET_KEY"
                        export AWS_DEFAULT_REGION="$AWS_REGION"

                        # Verify credentials
                        aws sts get-caller-identity

                        # Remove problematic files
                        find . -name "*:Zone.Identifier" -delete
                        rm -f .*.swp || true

                        # Sync without ACL
                        aws s3 sync . s3://$S3_BUCKET/ \\
                            --exclude ".git/*" \\
                            --exclude "Jenkinsfile" \\
                            --delete

                        # Invalidate CloudFront
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
        echo 'Post-build steps skipped for now.'
//            cleanWs()
        }
    }
