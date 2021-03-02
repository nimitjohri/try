def scmVars
pipeline {
    agent any
    tools {
        maven 'M3'
    }
    options {
        timestamps()
        timeout(time: 1, unit: 'HOURS')
        skipDefaultCheckout()
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
                    bat 'mvn sonar:sonar'
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
                        bat 'docker build -t nimit07/nagp-devops-try-dev:%BUILD_NUMBER% --no-cache -f Dockerfile .'
                    } else if (scmVars.GIT_BRANCH == "origin/prod") {
                        bat 'docker build -t nimit07/nagp-devops-try-prod:%BUILD_NUMBER% --no-cache -f Dockerfile .'
                    }
                }
            }
        }

        stage ('docker push') {
            steps {
                script {
                    bat 'docker login -u nimit07 -p Human@123'
                    if (scmVars.GIT_BRANCH == "origin/dev") {
                        bat 'docker push nimit07/nagp-devops-try-dev:%BUILD_NUMBER%'
                    } else if (scmVars.GIT_BRANCH == "origin/prod") {
                        bat 'docker push nimit07/nagp-devops-try-prod:%BUILD_NUMBER%'
                    }
                }
            }
        }

        stage ('Stop Running Contaiers') {
            steps {
                script {
                    if (scmVars.GIT_BRANCH == "origin/dev") {
                        bat '''
                        echo %tagname%
                        for /f %%i in ('docker ps -aqf "name=^nagp-devops-try-dev"') do set containerId=%%i
                        echo %containerId%
                        If "%containerId%" == "" (
                            echo "No running container"
                        ) else (
                            docker stop %containerId%
                            docker rm -f %containerId%
                        )
                        '''
                    } else if  (scmVars.GIT_BRANCH == "origin/prod") {
                        bat '''
                        echo %tagname%
                        for /f %%i in ('docker ps -aqf "name=^nagp-devops-try-prod"') do set containerId=%%i
                        echo %containerId%
                        If "%containerId%" == "" (
                            echo "No running container"
                        ) else (
                            docker stop %containerId%
                            docker rm -f %containerId%
                        )
                        '''
                    }
                }
            }
        }

        stage ('Docker Deployment') {
            steps {
                script {
                    if (scmVars.GIT_BRANCH == "origin/dev") {
                        bat 'docker run --name nagp-devops-try-dev -d -p 6310:8080 nimit07/nagp-devops-try-dev:%BUILD_NUMBER%'
                    } else if  (scmVars.GIT_BRANCH == "origin/prod") {
                        bat 'docker run --name nagp-devops-try-prod -d -p 6410:8080 nimit07/nagp-devops-try-prod:%BUILD_NUMBER%'
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
