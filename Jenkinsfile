pipeline {
    agent any

    environment {
        GIT_CREDENTIALS_ID = 'jenkins-assignment'
        GIT_REPO_URL = 'https://github.com/NadunOvitigala/jenkins-assignment.git'
        GIT_Branch = 'dev'
        SITE_URL = 'http://13.233.140.64/'
    }
    stages {
        stage ('sonarqube') {
            steps {
                script {
                    sh 'whoami'
                    sh '/opt/sonar-scanner/bin/sonar-scanner -Dsonar.projectKey=jenkins-assignment -Dsonar.host.url=http://localhost:9000 -Dsonar.login=sqp_524d836223a10583b80f2bc2bb83042ff7734a29'
                }
            }
        }
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
                    sshPublisher(publishers: [sshPublisherDesc(configName: 'ubuntu', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'sudo systemctl restart apache2.service', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'index.html')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
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