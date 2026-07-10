pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 45, unit: 'MINUTES')
        disableConcurrentBuilds()                 // avoids two builds racing on the same container/tag
        skipDefaultCheckout(true)                 // checkout happens explicitly in the Checkout stage
        buildDiscarder(logRotator(numToKeepStr: '30', artifactNumToKeepStr: '5'))
    }

    environment {
        IMAGE_NAME             = 'mishobo/todo-list-app'
        CONTAINER_NAME         = 'todo-app'
        APP_PORT               = '4567'
        DOCKERHUB_CREDENTIALS  = 'dockerhub-credentials'  // Jenkins Credentials ID: Docker Hub username + access token
        DEPLOY_SSH_CREDENTIALS = 'vm-deploy-ssh-key'      // Jenkins Credentials ID: SSH private key
        DEPLOY_USER            = 'deploy'
        DEPLOY_HOST            = 'your.vm.example.com'    // TODO: replace with the deployment VM hostname/IP
        NOTIFY_EMAIL           = 'team@example.com'       // TODO: replace with your real notification list
        GRADLE_OPTS            = '-Dorg.gradle.daemon=false'
        TRIVY_SEVERITY         = 'HIGH,CRITICAL'          // image scan fails the build on these severities
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    def branch = (env.BRANCH_NAME ?: env.GIT_BRANCH ?: 'unknown').replaceFirst(/^origin\//, '')
                    env.GIT_BRANCH_NAME = branch
                    env.IS_RELEASE_BRANCH = (branch == 'main' || branch == 'master').toString()
                    env.IMAGE_TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT_SHORT}"
                }
                // Fail fast with a readable message if the agent is missing required tooling.
                sh 'command -v docker >/dev/null || { echo "docker not found on agent"; exit 1; }'
                echo "Building ${env.GIT_BRANCH_NAME} @ ${env.GIT_COMMIT_SHORT} as ${env.IMAGE_NAME}:${env.IMAGE_TAG} (release branch: ${env.IS_RELEASE_BRANCH})"
            }
        }

        stage('Build') {
            steps {
                sh 'chmod +x ./gradlew'   // the exec bit is not guaranteed to survive checkout on every agent
                sh './gradlew clean build -x test --no-daemon'
            }
        }

        stage('Test') {
            steps {
                sh './gradlew test --no-daemon'
            }
            post {
                always {
                    // allowEmptyResults=false: a build where tests silently didn't run must fail, not pass
                    junit testResults: 'build/test-results/test/*.xml', allowEmptyResults: false
                }
            }
        }

        stage('Package') {
            steps {
                sh './gradlew installDist -x test --no-daemon'
                archiveArtifacts artifacts: 'build/install/java-todo/**', fingerprint: true
            }
        }

        stage('Containerize') {
            steps {
                sh '''
                    docker build \
                        --label "org.opencontainers.image.revision=${GIT_COMMIT_SHORT}" \
                        --label "org.opencontainers.image.source=${GIT_URL:-unknown}" \
                        -t "${IMAGE_NAME}:${IMAGE_TAG}" \
                        -t "${IMAGE_NAME}:build-${BUILD_NUMBER}" \
                        .
                '''
            }
        }

        stage('Image security scan') {
            // Shift-left gate: scan the image we just built on every branch, before it can be
            // pushed or deployed. Trivy runs from its official image so the agent only needs
            // Docker (no extra install); the named volume caches the vuln DB between builds.
            steps {
                sh '''
                    docker run --rm \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        -v trivy-cache:/root/.cache/ \
                        aquasec/trivy:latest image \
                            --scanners vuln \
                            --severity "${TRIVY_SEVERITY}" \
                            --ignore-unfixed \
                            --exit-code 1 \
                            --no-progress \
                            "${IMAGE_NAME}:${IMAGE_TAG}"
                '''
            }
        }

        stage('Push') {
            when { expression { env.IS_RELEASE_BRANCH == 'true' } }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: env.DOCKERHUB_CREDENTIALS,
                    usernameVariable: 'REGISTRY_USER',
                    passwordVariable: 'REGISTRY_TOKEN'
                )]) {
                    retry(2) {
                        sh '''
                            echo "$REGISTRY_TOKEN" | docker login -u "$REGISTRY_USER" --password-stdin
                            docker push "${IMAGE_NAME}:${IMAGE_TAG}"
                            docker tag "${IMAGE_NAME}:${IMAGE_TAG}" "${IMAGE_NAME}:latest"
                            docker push "${IMAGE_NAME}:latest"
                        '''
                    }
                }
            }
            post {
                always {
                    sh 'docker logout || true'
                }
            }
        }

        stage('Approve deploy') {
            // Manual production gate. Holds here (not an executor-heavy stage) until a human
            // approves or the timeout expires; the approver's name is recorded for the audit trail.
            when { expression { env.IS_RELEASE_BRANCH == 'true' } }
            options { timeout(time: 15, unit: 'MINUTES') }
            steps {
                script {
                    env.DEPLOY_APPROVER = input(
                        message: "Deploy ${env.IMAGE_NAME}:${env.IMAGE_TAG} to ${env.DEPLOY_HOST}?",
                        ok: 'Deploy',
                        submitterParameter: 'APPROVER'
                    )
                }
                echo "Deploy approved by ${env.DEPLOY_APPROVER}"
            }
        }

        stage('Deploy') {
            when { expression { env.IS_RELEASE_BRANCH == 'true' } }
            options { timeout(time: 10, unit: 'MINUTES') }
            steps {
                // Refuse to deploy against the placeholder host — prevents a misconfigured
                // job from silently doing nothing (or hitting the wrong machine).
                sh '''
                    if [ "${DEPLOY_HOST}" = "your.vm.example.com" ]; then
                        echo "DEPLOY_HOST is still the placeholder value — set the real host before deploying."
                        exit 1
                    fi
                '''
                sshagent(credentials: [env.DEPLOY_SSH_CREDENTIALS]) {
                    // accept-new trusts the host key on first contact only; pre-populate
                    // known_hosts on the Jenkins agent if you need strict pinning.
                    sh '''
                        ssh -o StrictHostKeyChecking=accept-new "${DEPLOY_USER}@${DEPLOY_HOST}" \
                            "IMAGE='${IMAGE_NAME}:${IMAGE_TAG}' CONTAINER='${CONTAINER_NAME}' PORT='${APP_PORT}' bash -s" <<'REMOTE'
set -euo pipefail

echo "Pulling $IMAGE ..."
docker pull "$IMAGE"

# Remember what is currently running so we can roll back if the new version is unhealthy.
PREVIOUS_IMAGE=$(docker inspect --format '{{.Config.Image}}' "$CONTAINER" 2>/dev/null || true)

docker rm -f "$CONTAINER" 2>/dev/null || true
docker run -d \
    --name "$CONTAINER" \
    --restart unless-stopped \
    -p "$PORT:$PORT" \
    "$IMAGE"

echo "Waiting for $CONTAINER to become healthy on port $PORT ..."
for i in $(seq 1 30); do
    if curl -fsS "http://localhost:$PORT/" >/dev/null 2>&1; then
        echo "Deploy OK: $IMAGE is serving traffic."
        exit 0
    fi
    sleep 2
done

echo "Health check FAILED for $IMAGE — last container logs:"
docker logs --tail 100 "$CONTAINER" || true
docker rm -f "$CONTAINER" || true

if [ -n "$PREVIOUS_IMAGE" ]; then
    echo "Rolling back to $PREVIOUS_IMAGE ..."
    docker run -d --name "$CONTAINER" --restart unless-stopped -p "$PORT:$PORT" "$PREVIOUS_IMAGE"
fi
exit 1
REMOTE
                    '''
                }
            }
        }
    }

    post {
        success {
            // Requires the Email Extension plugin (emailext).
            emailext(
                to: env.NOTIFY_EMAIL,
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER} (${env.GIT_BRANCH_NAME} @ ${env.GIT_COMMIT_SHORT})",
                body: """Pipeline succeeded.

Job:    ${env.JOB_NAME} #${env.BUILD_NUMBER}
Branch: ${env.GIT_BRANCH_NAME} @ ${env.GIT_COMMIT_SHORT}
Image:  ${env.IMAGE_NAME}:${env.IMAGE_TAG}
Logs:   ${env.BUILD_URL}console
"""
            )
        }
        failure {
            emailext(
                to: env.NOTIFY_EMAIL,
                subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER} (${env.GIT_BRANCH_NAME} @ ${env.GIT_COMMIT_SHORT})",
                // Declarative post{} does not reliably expose the failing stage name, so link to
                // the console/pipeline view rather than assert a stage that may be misleading.
                body: """Pipeline FAILED — see the log for the failing stage.

Job:    ${env.JOB_NAME} #${env.BUILD_NUMBER}
Branch: ${env.GIT_BRANCH_NAME} @ ${env.GIT_COMMIT_SHORT}
Logs:   ${env.BUILD_URL}console
"""
            )
        }
        always {
            // Drop this build's local image tags so the agent's disk doesn't fill up;
            // the pushed registry copies are the durable artifacts.
            sh '''
                docker rmi "${IMAGE_NAME}:${IMAGE_TAG}" "${IMAGE_NAME}:build-${BUILD_NUMBER}" 2>/dev/null || true
                docker image prune -f >/dev/null 2>&1 || true
            '''
            cleanWs(deleteDirs: true, notFailBuild: true)
        }
    }
}
