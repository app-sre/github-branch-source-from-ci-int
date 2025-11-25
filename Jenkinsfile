// Jenkinsfile for gh-pr-and-build template
// See: https://www.jenkins.io/doc/book/pipeline/jenkinsfile/

pipeline {
    // Agent configuration - defines where the pipeline runs
    agent {
        node {
            // Use spot instances for cost efficiency
            label 'rhel8-spot'
        }
    }

    // Pipeline options
    options {
        // Add timestamps to console output
        timestamps()
    }

    stages {
        // Stage 1: PR Check - runs for pull requests only
        stage('PR Check') {
            when {
                // Only execute when building a pull request
                // Environment variables available: CHANGE_ID, CHANGE_AUTHOR, CHANGE_TARGET, etc.
                changeRequest()
            }
            steps {
                // Run PR validation script
                sh './pr_check.sh'
            }
        }

        // Stage 2: Build - runs for main branch only
        stage('Build') {
            when {
                // Only execute when building the main branch
                branch 'main'
            }
            steps {
                // VaultBuildWrapper injects secrets as environment variables
                // Secrets are ONLY available in this stage, not in PR Check for security
                wrap([$class: 'VaultBuildWrapper',
                    vaultSecrets: [
                        [
                            // Vault path containing the secrets
                            path: 'app-sre/quay/app-sre-push',
                            secretValues: [
                                // Map Vault keys to environment variables
                                [envVar: 'QUAY_USER', vaultKey: 'user'],
                                [envVar: 'QUAY_TOKEN', vaultKey: 'token']
                            ]
                        ]
                    ]
                ]) {
                    // Run build/deploy script with access to secrets
                    sh './build_deploy.sh'
                }
            }
        }
    }

    // Post-build actions
    post {
        always {
            // Clean workspace after every build to save disk space
            cleanWs()
        }
    }
}
