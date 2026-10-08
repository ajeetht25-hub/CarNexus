pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(false)
    }

    environment {
        APP_NAME = 'CarNexus'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out ${APP_NAME}..."

                checkout scm
            }
        }

        stage('Inspect Project') {
            steps {
                sh '''
                    echo "===== Project Structure ====="
                    pwd
                    echo ""
                    find . -maxdepth 2 -type f | sort | head -200
                    echo ""
                    echo "===== PHP Version ====="
                    php --version || true
                    echo ""
                    echo "===== Composer Version ====="
                    composer --version || true
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    set -e

                    if [ -f composer.json ]; then
                        echo "composer.json found."
                        composer install \
                            --no-interaction \
                            --prefer-dist \
                            --optimize-autoloader
                    else
                        echo "No composer.json found. Skipping Composer."
                    fi
                '''
            }
        }

        stage('Validate PHP') {
            steps {
                sh '''
                    set -e

                    PHP_FILES=$(find . -type f -name "*.php" \
                        -not -path "./vendor/*")

                    if [ -n "$PHP_FILES" ]; then
                        echo "Checking PHP syntax..."

                        for file in $PHP_FILES; do
                            php -l "$file"
                        done
                    else
                        echo "No PHP files found."
                    fi
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    set -e

                    if [ -f vendor/bin/phpunit ]; then
                        echo "Running PHPUnit..."
                        vendor/bin/phpunit
                    elif [ -f phpunit.xml ] || [ -f phpunit.xml.dist ]; then
                        echo "PHPUnit configuration found."
                        ./vendor/bin/phpunit
                    else
                        echo "No PHPUnit tests configured. Skipping tests."
                    fi
                '''
            }
        }

        stage('Build Artifact') {
            steps {
                sh '''
                    set -e

                    rm -rf build
                    mkdir -p build

                    echo "Creating deployment artifact..."

                    rsync -av \
                        --exclude='.git' \
                        --exclude='.gitignore' \
                        --exclude='Jenkinsfile' \
                        --exclude='build' \
                        --exclude='tests' \
                        ./ build/

                    echo "Artifact created successfully."

                    echo "===== Build Contents ====="
                    find build -maxdepth 3 -type f | sort | head -200
                '''
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'build/**',
                                 fingerprint: true,
                                 allowEmptyArchive: false
            }
        }
    }

    post {

        success {
            echo "${APP_NAME} build completed successfully."
        }

        failure {
            echo "${APP_NAME} build FAILED."
        }

        always {
            echo "Cleaning workspace..."
            cleanWs(
                deleteDirs: true,
                disableDeferredWipeout: true
            )
        }
    }
}
