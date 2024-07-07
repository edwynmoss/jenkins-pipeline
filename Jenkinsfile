pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-hub'
        IMAGE = 'your-docker-image:latest'
        NVD_API_KEY = 'your-nvd-api-key'
        BUILD_START_TIME = "${new Date().time}"
        BUILD_END_TIME = "${new Date().time}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('OWASP Dependency-Check Vulnerabilities') {
            steps {
                script {
                    def additionalArgs = [
                        '-o', './',
                        '-s', './',
                        '-f', 'ALL',
                        '--prettyPrint',
                        '--nvdApiKey', env.NVD_API_KEY
                    ].join(' ')

                    try {
                        dependencyCheck additionalArguments: additionalArgs, odcInstallation: 'OWASP Dependency-Check Vulnerabilities'
                        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
                    } catch (Exception e) {
                        echo "Dependency-Check Failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    try {
                        // Set build start time
                        env.BUILD_START_TIME = "${new Date().time}"

                        // Building Docker image
                        def DockerImage = docker.build("${env.IMAGE}", "-f Dockerfile .")

                        // Pushing Docker image to Docker Hub
                        docker.withRegistry('https://registry-1.docker.io/v2/', "${env.DOCKER_CREDENTIALS_ID}") {
                            DockerImage.push()
                        }

                        // Set build end time
                        env.BUILD_END_TIME = "${new Date().time}"
                    } catch (Exception e) {
                        echo "Build and Push Failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }

        stage('Trigger Deployment') {
            steps {
                script {
                    try {
                        // SSH deployment
                        def remote = [:]
                        remote.name = 'staging'
                        remote.host = 'xxx.xxx.xxx.xxx' // Replace with your staging server IP
                        remote.user = 'your-ssh-user' // Replace with your SSH username
                        remote.password = 'your-ssh-password' // Replace with your SSH password
                        remote.allowAnyHosts = true

                        sshCommand(remote: remote, command: "docker pull ${env.IMAGE}")
                        sshCommand(remote: remote, command: "docker compose -f /path/to/compose.yaml up -d --remove-orphans")
                    } catch (Exception e) {
                        echo "Deployment Failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                sendDiscordNotification("Build and Deployment Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}", 3066993)
            }
        }
        failure {
            script {
                sendDiscordNotification("Build and Deployment Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}", 15158332)
            }
        }
    }
}

def sendDiscordNotification(message, color) {
    def url = 'https://discord.com/api/webhooks/your-webhook-id/your-webhook-token' // Replace with your Discord webhook URL
    def headers = [
        'Content-Type': 'application/json'
    ]
    
    def body = [
        username: 'Pipeline Bot',
        embeds: [
            [
                title: 'Deployment Status',
                description: message,
                color: color,
                fields: [
                    [
                        name: 'Status',
                        value: currentBuild.result ?: 'N/A',
                        inline: true
                    ],
                    [
                        name: 'Build Number',
                        value: "#${env.BUILD_NUMBER}",
                        inline: true
                    ],
                    [
                        name: 'Build Duration',
                        value: currentBuild.durationString ?: 'N/A',
                        inline: true
                    ],
                    [
                        name: 'Start Time',
                        value: new Date(Long.parseLong(env.BUILD_START_TIME ?: '0')).format("yyyy-MM-dd HH:mm:ss"),
                        inline: true
                    ],
                    [
                        name: 'End Time',
                        value: new Date(Long.parseLong(env.BUILD_END_TIME ?: '0')).format("yyyy-MM-dd HH:mm:ss"),
                        inline: true
                    ],
                    [
                        name: 'Build URL',
                        value: "${env.BUILD_URL}",
                        inline: false
                    ]
                ],
                timestamp: new Date().format("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'"),
                footer: [
                    text: 'Copyright Your Company',
                    icon_url: 'https://example.com/path/to/icon.png' // Replace with your icon URL
                ]
            ]
        ]
    ]
    
    def jsonBody = new groovy.json.JsonBuilder(body).toString()
    
    try {
        def response = httpRequest(
            acceptType: 'APPLICATION_JSON',
            contentType: 'APPLICATION_JSON',
            httpMode: 'POST',
            requestBody: jsonBody,
            url: url,
            headers: headers
        )

        echo "Discord notification sent successfully. Response: ${response}"
    } catch (Exception e) {
        echo "Failed to send Discord notification: ${e.message}"
    }
}
