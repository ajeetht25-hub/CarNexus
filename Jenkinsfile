pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out CarNexus...'

                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                sh '''
                    set -e

                    echo "===== PHP ====="
                    php --version

                    echo ""
                    echo "===== Git ====="
                    git --version

                    echo ""
                    echo "===== Project ====="
                    pwd
                    ls -la
                '''
            }
        }

        stage('Validate PHP') {
            steps {
                sh '''
                    set -e

                    echo "Checking PHP files..."

                    find Backend -type f -name "*.php" -print

                    echo ""
                    echo "Running PHP syntax checks..."

                    find Backend -type f -name "*.php" -print0 | while IFS= read -r -d "" file
                    do
                        echo "Checking: $file"
                        php -l "$file"
                    done

                    echo ""
                    echo "All PHP files passed syntax validation."
                '''
            }
        }

        stage('Validate Frontend') {
            steps {
                sh '''
                    set -e

                    echo "Checking frontend files..."

                    test -f index.html
                    test -d Frontend
                    test -f Frontend/style.css
                    test -f Frontend/web.js

                    echo "Frontend files found successfully."
                '''
            }
        }

        stage('Validate Database Schema') {
            steps {
                sh '''
                    set -e

                    if [ -f Toyota_schema.sql ]; then
                        echo "Database schema found:"
                        ls -lh Toyota_schema.sql
                    else
                        echo "WARNING: Toyota_schema.sql not found."
                    fi
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    set -e

                    rm -rf build
                    mkdir -p build

                    echo "Creating build artifact..."

                    rsync -av \
                        --exclude=".git" \
                        --exclude="build" \
                        --exclude="Jenkinsfile" \
                        ./ build/

                    echo ""
                    echo "Build completed successfully."
                '''
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts(
                    artifacts: 'build/**',
                    fingerprint: true,
                    allowEmptyArchive: false
                )
            }
        }
    }

    post {
        success {
            echo '======================================'
            echo ' CarNexus BUILD SUCCESSFUL'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo ' CarNexus BUILD FAILED'
            echo '======================================'
        }

        always {
            cleanWs(
                deleteDirs: true,
                disableDeferredWipeout: true
            )
        }
    }
}
