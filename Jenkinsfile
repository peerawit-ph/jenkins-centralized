pipeline {
    agent any

    stages {
        stage('Extract Repository Info') {
            steps {
                script {
                    // Extract repository name from GIT_URL
                    if (env.GIT_URL) {
                        def repoUrl = env.GIT_URL
                        def repoName = repoUrl.split('/').last().replaceAll('\\.git$', '')
                        env.REPO_NAME = repoName
                    } else {
                        error("GIT_URL environment variable is not set")
                    }

                    // Use GIT_BRANCH as environment, filter out origin/ prefix, default to 'main' if not set
                    def branchName = env.GIT_BRANCH ?: 'main'
                    env.ENVIRONMENT = branchName.replaceAll('^origin/', '')

                    // Construct the path to the specific Jenkinsfile
                    env.TARGET_JENKINSFILE = "${env.REPO_NAME}/${env.ENVIRONMENT}.jenkinsfile"

                    echo "=== Dispatcher Information ==="
                    echo "GIT_URL: ${env.GIT_URL}"
                    echo "GIT_BRANCH: ${env.GIT_BRANCH}"
                    echo "Extracted Repository Name: ${env.REPO_NAME}"
                    echo "Environment: ${env.ENVIRONMENT}"
                    echo "Target Jenkinsfile: ${env.TARGET_JENKINSFILE}"
                    echo "============================="
                }
            }
        }

        stage('Validate Target Jenkinsfile') {
            steps {
                script {
                    if (!fileExists(env.TARGET_JENKINSFILE)) {
                        error("Target Jenkinsfile does not exist: ${env.TARGET_JENKINSFILE}")
                    }
                    echo "Target Jenkinsfile found: ${env.TARGET_JENKINSFILE}"
                }
            }
        }

        stage('Load and Execute Target Pipeline') {
            steps {
                script {
                    echo "Loading pipeline from: ${env.TARGET_JENKINSFILE}"

                    // Load and execute the target Jenkinsfile
                    load env.TARGET_JENKINSFILE
                }
            }
        }
    }

    post {
        always {
            script {
                echo "=== Pipeline Execution Summary ==="
                echo "Repository: ${env.REPO_NAME}"
                echo "Environment: ${env.ENVIRONMENT}"
                echo "Jenkinsfile Used: ${env.TARGET_JENKINSFILE}"
                echo "Status: ${currentBuild.result ?: 'SUCCESS'}"
                echo "================================="
            }
        }

        failure {
            script {
                echo "Pipeline failed for repository: ${env.REPO_NAME}, environment: ${env.ENVIRONMENT}"
                echo "Check the target Jenkinsfile: ${env.TARGET_JENKINSFILE}"
            }
        }
    }
}
