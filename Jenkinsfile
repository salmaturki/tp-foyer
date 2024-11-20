def projectDockerImage
pipeline {
    agent any

    environment {
        dockerImageName = "5ds7-devops-salma"
        dockerUsername = "salmaturki"
    }

    stages {
        stage('Clean ws'){
            steps{
                script{
                    cleanWs()
                }
            }
        }
        stage('Github checkout'){
            steps{
                git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/salmaturki/tp-foyer'
            }
        }

        stage('Compile Backend') {
            steps {
                sh "mvn clean compile"
            }
        }

        stage('Sonarqube Analysis'){
            steps{
                withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonar') {
                    sh '''  mvn sonar:sonar -Dsonar.projectKey=DevopsProject'''
            }}
        }

        stage('Nexus publish'){
            steps{
                script {
                    sh 'mvn deploy -DskipTests'
                }
            }
        }

        stage('Build docker images'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                        projectDockerImage = docker.build dockerUsername+"/"+dockerImageName + ":latest", " ."
                    }
                }
            }
        }

        stage('Docker push images'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                        projectDockerImage.push()
                    }
                }
            }
        }

        stage('Run and monitor app'){
            steps{
                script{
                    sh "docker compose up -d"
                }
            }
        }

    }

        post{
            always{
                script{
                        def pipelineStatus = currentBuild.result ?: "UNKNOWN"
                        def bannerColor = pipelineStatus.toUpperCase() == "SUCCESS" ? "#28a745" : "#dc3545"
                        def jobName = env.JOB_NAME
                        def buildNumber = env.BUILD_NUMBER
                        def htmlContent = """
                            <html>
                                <body>
                                    <div style="padding: 10px; color: white; background-color: ${bannerColor}; text-align: center; font-size: 24px;">
                                        ${pipelineStatus}
                                    </div>
                                    <p>Hello,</p>
                                    <p>The Jenkins job has completed with status: <strong>${pipelineStatus}</strong>.</p>
                                    <p>Check the details on your Jenkins server.</p>
                                </body>
                            </html>
                        """

                        emailext (
                            subject: "${jobName} - ${buildNumber} : ${pipelineStatus}",
                            body: htmlContent,
                            mimeType: 'text/html',
                            to: PERSONAL_EMAIL,
                        )
                }
            }
        }
}