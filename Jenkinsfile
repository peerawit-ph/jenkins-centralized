pipeline {
    agent any

    stages {
        stage('Extract Repository Info') {
            steps {
                script {
                    echo "${WORKSPACE}"
                    // Extract repository name from GIT_URL
                    if (env.GIT_URL) {
                        def repoUrl = env.GIT_URL
                        def fullRepoName = repoUrl.split('/').last().replaceAll('\\.git$', '')
                        // Handle jenkins-centralized -> jenkins-lab mapping
                        def repoName = fullRepoName.replace('jenkins-centralized', 'jenkins-lab')
                        env.REPO_NAME = repoName
                        echo "DEBUG: Full repo name: ${fullRepoName}"
                        echo "DEBUG: Mapped repo name: ${repoName}"
                    } else {
                        error("GIT_URL environment variable is not set")
                    }

                    // Use GIT_BRANCH as environment, filter out origin/ prefix, default to 'main' if not set
                    def branchName = env.GIT_BRANCH ?: 'main'
                    env.ENVIRONMENT = branchName.replaceAll('^origin/', '')
                    echo "DEBUG: Original branch: ${env.GIT_BRANCH}"
                    echo "DEBUG: Cleaned environment: ${env.ENVIRONMENT}"

                    // Construct the path to the specific Jenkinsfile
                    env.TARGET_JENKINSFILE = "${env.REPO_NAME}/${env.ENVIRONMENT}.jenkinsfile"
                    echo "DEBUG: Target path: ${env.TARGET_JENKINSFILE}"
                    echo "DEBUG: Workspace: ${env.WORKSPACE}"

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

        stage('Debug Workspace') {
            steps {
                script {
                    echo "=== Workspace Contents ==="
                    sh 'pwd'
                    sh 'ls -la'
                    sh 'find . -name "*.jenkinsfile" -type f'
                    echo "=========================="
                }
            }
        }

        stage('Validate Target Jenkinsfile') {
            steps {
                script {
                    echo "${WORKSPACE}"
                    sh 'ls -ltr'
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
