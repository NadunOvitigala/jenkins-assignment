pipeline {
    agent any

    environment {
        GIT_CREDENTIALS_ID = 'jenkins-assignment'
        GIT_REPO_URL = 'https://github.com/NadunOvitigala/jenkins-assignment.git'
        GIT_Branch = 'dev'
        SITE_URL = 'http://3.110.148.253/'
    }
    stages {
        stage ('Upload Artifacts') {
            steps {
                script {
                    s3Upload consoleLogLevel: 'INFO', dontSetBuildResultOnFailure: false, dontWaitForConcurrentBuildCompletion: false, entries: [[bucket: 'jenkins-assignment', excludedFile: '', flatten: false, gzipFiles: false, keepForever: false, managedArtifacts: true, noUploadOnFailure: false, selectedRegion: 'ap-south-1', showDirectlyInBrowser: false, sourceFile: '', storageClass: 'STANDARD', uploadFromSlave: false, useServerSideEncryption: false]], pluginFailureResultConstraint: 'FAILURE', profileName: 's3-full', userMetadata: []                
                }
            }
        }
        stage ('index.html') {
            steps {
                script {
                    git branch:env.GIT_Branch, credentialsId: env.GIT_CREDENTIALS_ID, url: env.GIT_REPO_URL
                }
            }
        }
        stage ('SSH to Apache Server') {
            steps {
                script {
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        sh 'ssh -o StrictHostKeyChecking=no your-username@your-server-ip "your-bash-command"'
                    }
                }
            }
        }
        stage ('restart Apache') {
            steps {
                script {
                    sh 'sudo systemctl restart apache2.service'
                }
            }
        }
        stage ('Check Apache Status') {
            steps {
                script {
                    def response = sh(script: "curl -o /dev/null -s -w \"%{http_code}\" ${env.SITE_URL}", returnStdout: true).trim()
                    if (response == '200') {
                        echo 'Apache is running successfully with status code 200'
                        currentBuild.result = 'SUCCESS'
                    } else {
                        echo "Apache is not running successfully. Status code: ${response}"
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
    }
}