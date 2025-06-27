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

                # Verify credentials
                aws sts get-caller-identity

                # Remove problematic files
                find . -name "*:Zone.Identifier" -delete
                rm -f .*.swp || true

                # Sync without ACL
                aws s3 sync . s3://$S3_BUCKET/ \
                    --exclude ".git/*" \
                    --exclude "Jenkinsfile" \
                    --delete

                # Invalidate CloudFront
                aws cloudfront create-invalidation \
                    --distribution-id $CF_DIST_ID \
                    --paths "/*"
            '''
        }
    }
}

