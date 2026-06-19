pipeline {
    agent any

    environment {
        IMAGE_NAME = 'sentiment-ai'
        REGISTRY = 'ghcr.io/satlix'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git log --oneline -5'
                sh 'echo "IMAGE_TAG=$(git rev-parse --short HEAD)"'
            }
        }

        stage('Lint') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:lint .
                    docker run --rm ${IMAGE_NAME}:lint sh -c "pip install flake8 -q && flake8 src/ --max-line-length=100"
                '''
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                    IMAGE_TAG=$(git rev-parse --short HEAD)
                    echo "Building image with tag: ${IMAGE_TAG}"

                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .

                    docker rm -f test-runner 2>/dev/null || true
                    set +e
                    docker run \
                        -e CI=true \
                        --name test-runner \
                        ${IMAGE_NAME}:${IMAGE_TAG} \
                        pytest tests/ -v \
                        --cov=src \
                        --cov-report=xml:/tmp/coverage.xml \
                        --cov-report=term-missing \
                        --cov-fail-under=70
                    TEST_EXIT_CODE=$?
                    set -e

                    docker cp test-runner:/tmp/coverage.xml ./coverage.xml 2>/dev/null || true
                    docker rm -f test-runner 2>/dev/null || true

                    exit $TEST_EXIT_CODE
                '''
            }
            post {
                failure {
                    echo 'Tests echoues ou coverage insuffisant (< 70%)'
                }
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONARQUBE_TOKEN = credentials('sonar-token')
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        docker run --rm \
                            --network cicd-network \
                            --volumes-from jenkins \
                            -w "$WORKSPACE" \
                            -e SONAR_HOST_URL="$SONAR_HOST_URL" \
                            -e SONAR_TOKEN="$SONARQUBE_TOKEN" \
                            sonarsource/sonar-scanner-cli:latest \
                            sonar-scanner \
                            -Dsonar.projectKey=sentiment-ai \
                            -Dsonar.projectName=SentimentAI \
                            -Dsonar.projectBaseDir="$WORKSPACE" \
                            -Dsonar.sources=src \
                            -Dsonar.python.version=3.11 \
                            -Dsonar.python.coverage.reportPaths=coverage.xml \
                            -Dsonar.sourceEncoding=UTF-8 \
                            -Dsonar.scanner.metadataFilePath=$WORKSPACE/report-task.txt
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-token',
                    usernameVariable: 'REGISTRY_USER',
                    passwordVariable: 'REGISTRY_PASS'
                )]) {
                    sh '''
                        IMAGE_TAG=$(git rev-parse --short HEAD)
                        echo "Pushing image with tag: ${IMAGE_TAG}"

                        echo "${REGISTRY_PASS}" | docker login ghcr.io -u "${REGISTRY_USER}" --password-stdin

                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${REGISTRY}/${IMAGE_NAME}:latest
                        docker push ${REGISTRY}/${IMAGE_NAME}:latest
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker compose down -v 2>/dev/null || true'
        }
        success {
            sh '''
                IMAGE_TAG=$(git rev-parse --short HEAD)
                echo "Pipeline reussi ! Image : ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
            '''
        }
        failure {
            echo 'Pipeline echoue. Consultez les logs ci-dessus.'
        }
    }
}