def scmVars
pipeline {
    agent any
    tools {
        maven 'M3'
    }
    options {
        timestamps()
        timeout(time: 1, unit: 'HOURS')
        shikDefaultCheckout()
        buildDiscarder(logRotator(daysToKeepStr: '10', numToKeepStr: '10'))
    }
    stages {
        stage('checkout') {
            steps {
                script {
                    scmVars = checkout scm
                    echo scmVars.GIT_BRANCH
                }
            }
        }

        stage('Buid') {
            steps {
                script {
                    bat 'mvn clean install'
                }
            }
        }

        stage('Unit') {
            steps {
                script {
                    bat 'mvn test'
                }
            }
        }
        stage('Sonar') {
            steps {
                withSonarQubeEnv('SonarQube 8.4') {
                    bat 'mvn sonar: sonar'
                }
            }
        }

        stage('artifactory') {
            steps {
                script {
                    rtMavenDeployer(
                        id: "dev",
                        serverId: "artifactory 6.20",
                        snapshotRepo: "nagp-devops-exam-try-1",
                        releaseRepo: "nagp-devops-exam-try-1"
                    )

                    rtMavenRun(
                        pom: 'pom.xml',
                        goals: 'clean install',
                        deployerId: 'dev'
                    )

                    rtPublishBuildInfo(
                        serverId: "artifactory 6.20"
                    )
                }
            }
        }

        stage('Docker build') {
            steps {
                script {
                    if (scmVars.GIT_BRANCH == "origin/dev") {
                        bat 'docker build -t nimit07/nagp-devops-try-dev --no-cache -f Dockerfile .'
                    }
                }
            }
        }
    }
    post {
        always {
            junit 'target/surefire-reports/*.xml'
        }
        success {
            script {
                stage('Build succes') {
                    mail bcc: '',
                    body: """
                    Name: ${env.JOB_NAME}
                    BUILD: ${env.BUILD_NUMBER}
                    URL: ${env.BUILD_URL}
                    """,
                    subject: "Success: ${env.JOB_NAME}",
                    mimeType: "text/html",
                    from: "Jenkinserver",
                    charset: "utf-8",
                    to: "nimit.johri@nagarro.com"
                }
            }
        }

        failure {
            script {
                stage('Build failed') {
                    mail bcc: '',
                    body: """
                    Name: ${env.JOB_NAME}
                    BUILD: ${env.BUILD_NUMBER}
                    URL: ${env.BUILD_URL}
                    """,
                    subject: "Failed: ${env.JOB_NAME}",
                    mimeType: "text/html",
                    from: "Jenkinserver",
                    charset: "utf-8",
                    to: "nimit.johri@nagarro.com"
                }
            }
        }

    }
}
