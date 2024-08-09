pipeline{
    agent {
        label 'AGENT-1'
    }
    options{
        timeout(time: 30,unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment{
        def appVersion = ''
        nexusUrl = 'nexus.muvva.online:8081'
    }
    stages{
        stage('read the version'){
            steps{
                script{
                    def packageJSON = readJSON file: 'package.json'
                    appVersion = packageJSON.version
                    echo "app version is: $appVersion"
                }
            }
        }
        
         stage('building artifact'){
            steps{
                sh """
                   zip -q -r frontend-${appVersion}.zip * -x Jenkinsfile -x frontend-${appVersion}.zip
                   ls -ltr
                """
            }
        }
        stage('uploading artifact'){
            steps{
               script{
                nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: 'http',
                nexusUrl: "${nexusUrl}",
                groupId: 'com.expense',
                version: "${appVersion}",
                repository: "frontend",
                credentialsId: 'nexus-auth',
                artifacts: [
                    [artifactId: "frontend",
                    classifier: '',
                    file: 'frontend-' + "${appVersion}" + '.zip',
                    type: 'zip']
                ]
            )
               }
            }
        }
        stage('Deploy')
        {
            steps{
                script{
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}")
                    ]
                    build job: "frontend-deploy",
                    parameters: params , wait: false
                }
            }
        }
     }
        post{
            always{
                echo "Running always"
                deleteDir()
            }
        }
    }