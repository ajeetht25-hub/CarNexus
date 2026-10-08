pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                echo '========== CHECKOUT =========='

                git branch: 'main',
                    url: 'https://github.com/ajeetht25-hub/CarNexus.git'
            }
        }

        stage('Environment Check') {
            steps {
                sh '''
                    set -e

                    echo "========== ENVIRONMENT =========="

                    echo "PHP:"
                    php --version

                    echo ""
                    echo "Git:"
                    git --version

                    echo ""
                    echo "Workspace:"
                    pwd

                    echo ""
                    echo "Project:"
                    ls -la
                '''
            }
        }

        stage('PHP Validation') {
            steps {
                sh '''
                    set -e

                    echo "========== PHP VALIDATION =========="

                    if [ ! -d Backend ]; then
                        echo "ERROR: Backend directory not found"
                        exit 1
                    fi

                    COUNT=0

                    for file in Backend/*.php
                    do
                        if [ -f "$file" ]; then
                            COUNT=$((COUNT + 1))

                            echo "Checking: $file"

                            php -l "$file"
                        fi
                    done

                    echo ""
                    echo "Validated $COUNT PHP files."
                '''
            }
        }

        stage('Frontend Validation') {
            steps {
                sh '''
                    set -e

                    echo "========== FRONTEND VALIDATION =========="

                    echo "Checking index.html..."
                    test -f index.html

                    echo "Checking Frontend directory..."
                    test -d Frontend

                    echo "Checking CSS..."
                    test -f Frontend/style.css

                    echo "Checking JavaScript..."
                    test -f Frontend/web.js

                    echo ""
                    echo "Frontend validation successful."
                '''
            }
        }

        stage('Database Validation') {
            steps {
                sh '''
                    set -e

                    echo "========== DATABASE VALIDATION =========="

                    if [ -f Toyota_schema.sql ]; then
                        echo "Toyota_schema.sql found."
                        echo "Size:"
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

                    echo "========== BUILD =========="

                    rm -rf build
                    mkdir -p build

                    echo "Copying project files..."

                    rsync -av \
                        --exclude=".git" \
                        --exclude="build" \
                        --exclude="Jenkinsfile" \
                        ./ build/

                    echo ""
                    echo "Build completed successfully."

                    echo ""
                    echo "Build size:"
                    du -sh build
                '''
            }
        }

        stage('Archive') {
            steps {
                echo '========== ARCHIVE =========='

                archiveArtifacts(
                    artifacts: 'build/**',
                    fingerprint: true,
                    allowEmptyArchive: false
                )

                echo 'Build artifact archived successfully.'
            }
        }
    }

    post {

        success {
            echo '''
========================================
       CARNEXUS BUILD SUCCESSFUL
========================================
'''
        }

        failure {
            echo '''
========================================
         CARNEXUS BUILD FAILED
========================================
'''
        }

        always {
            echo 'Cleaning workspace...'

            cleanWs(
                deleteDirs: true,
                disableDeferredWipeout: true
            )
        }
    }
}
