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
                withCredentials([
                    string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY'),
                    string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_KEY')
                ]) {
                    sh '''
                        export AWS_ACCESS_KEY_ID="$AWS_ACCESS_KEY"
                        export AWS_SECRET_ACCESS_KEY="$AWS_SECRET_KEY"
                        export AWS_DEFAULT_REGION="$AWS_REGION"

                        echo "Verify credentials:"
                        aws sts get-caller-identity

                        echo "Clean up unnecessary files..."
                        find . -name "*:Zone.Identifier" -delete
                        rm -f .*.swp || true

                        echo "Uploading files to S3..."
                        aws s3 sync . s3://$S3_BUCKET/ \
                            --exclude ".git/*" \
                            --exclude "Jenkinsfile" \
                            --delete

                        echo "Invalidating CloudFront cache..."
                        aws cloudfront create-invalidation \
                            --distribution-id $CF_DIST_ID \
                            --paths "/*"
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'Post-build steps skipped for now.'
        }
    }
} // <--- This closing brace was missing

