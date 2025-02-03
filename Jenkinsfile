pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Tunexopez/realrepo.git'
            }
        }

        stage('Build Services') {
            steps {
                sh "docker-compose -f ${DOCKER_COMPOSE_FILE} build"
            }
        }

        stage('Start Services') {
            steps {
                sh "docker-compose -f ${DOCKER_COMPOSE_FILE} up -d"
            }
        }

        stage('Verify Health Checks') {
            steps {
                script {
                    def maxRetries = 5
                    def retryCount = 0
                    def healthy = false
                    
                    while (retryCount < maxRetries) {
                        def redisHealth = sh(script: "bash -c 'docker inspect --format={{.State.Health.Status}} \$(docker-compose -f ${DOCKER_COMPOSE_FILE} ps -q redis)'", returnStdout: true).trim()
                        def dbHealth = sh(script: "bash -c 'docker inspect --format={{.State.Health.Status}} \$(docker-compose -f ${DOCKER_COMPOSE_FILE} ps -q db)'", returnStdout: true).trim()
                        
                        if (redisHealth == "healthy" && dbHealth == "healthy") {
                            echo "✅ All services are healthy!"
                            healthy = true
                            break
                        }

                        echo "⏳ Waiting for services to be healthy... Attempt ${retryCount + 1}/${maxRetries}"
                        sleep 10
                        retryCount++
                    }

                    if (!healthy) {
                        error "❌ Services did not become healthy in time!"
                    }
                }
            }
        }

        stage('Run Seed Data (Optional)') {
            when {
                expression { params.RUN_SEED }
            }
            steps {
                sh "docker-compose -f ${DOCKER_COMPOSE_FILE} --profile seed up -d"
            }
        }

        stage('Clean Up') {
            steps {
                sh "docker-compose -f ${DOCKER_COMPOSE_FILE} down"
            }
        }
    }

    parameters {
        booleanParam(name: 'RUN_SEED', defaultValue: false, description: 'Run the seed data container')
    }
}
