def call(Map configMap){
    pipeline {
        agent {
            label 'AGENT-1'
        }
        options {
            timeout(time: 30, unit: 'MINUTES')
            disableConcurrentBuilds()
            ansiColor('xterm')
        }
        environment{
            def appVersion = '' //variable declaration
            nexusUrl = pipelineGlobals.nexusURL()
            region = pipelineGlobals.region()
            account_id = pipelineGlobals.account_id()
            component = configMap.get("component")
            project = configMap.get("project")
            def releaseExists = ""
        }
        stages {
            stage('read the version'){
                steps{
                    script{
                        echo sh(returnStdout: true, script: 'env')
                        def packageJson = readJSON file: 'package.json'
                        appVersion = packageJson.version
                        echo "application version: $appVersion"

                    }
                }
            }
            stage('Install Dependencies') {
                steps {
                sh """
                    npm install
                    ls -ltr
                    echo "application version: $appVersion"
                """
                }
            }
            stage('Build'){
                steps{
                    sh """
                    zip -q -r ${component}-${appVersion}.zip * -x Jenkinsfile -x ${component}-${appVersion}.zip
                    ls -ltr
                    """
                }
            }
            stage('Docker build'){
                steps{

                    sh """
                        aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com
                        docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/${project}-${component}:${appVersion} .
OBOBOBOBOBOB                        docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/${project}-${component}:${appVersion}
OBOBOBOBOBOBOBOBOBOBOB                    """
                }
OB            }
OB
            stage('Deploy'){
                steps{
                    script{
OB                        releaseExists = sh(script: "helm list -A --short | grep -w ${component} || true", returnStdout: true).trim()
                        if(releaseExists.isEmpty()){
                            echo "${component} not installed yet, first time installation"
                            sh"""
                                aws eks update-kubeconfig --region ${region} --name ${project}-dev
OBOBOBOBOBOBOBOBOB                                cd helm
OBOBOBOBOBOBOBOBOBOBOBOBOB                                sed -i 's/IMAGE_VERSION/${appVersion}/g' values.yaml
OBOBOBOBOBOBOBOB                                helm install ${component} -n ${project} .
OBOBOBOB                            """
OB                        }
OBOB                        else{
                            echo "${component} exists, running upgrade"
OB                            sh"""
                                aws eks update-kubeconfig --region ${region} --name ${project}-dev
OBOBOBOBOBOBOBOBOBOBOBOBOB                                cd helm
OBOBOBOBOBOBOBOBOBOBOB                                sed -i 's/IMAGE_VERSION/${appVersion}/g' values.yaml
OBOBOB                                helm upgrade ${component} -n ${project} .
OBOBOBOBOBOBOBOBOBOB                            """
OBOBOBOBOBOBOBOBOBOBOBOBOBOB                        }
OBOBOBOBOBOBOBOBOBOB                    }
OBOBOBOBOBOBOBOBOB                }
OBOB            }
OBOBOBOBOBOBOBOBOB            stage('Verify Deployment'){
OBOBOB                steps{
OBOBOBOBOBOBOBOBOBOB                    script{
OBOBOBOB                        rollbackStatus = sh(script: "kubectl rollout status deployment/backend -n ${project} --timeout=1m || true", returnStdout: true).trim()
OBOBOB                        if(rollbackStatus.contains('successfully rolled out')){
OBOBOBOBOBOBOBOBOBOBOBOBOBOBOBOBOBOBOB                            echo "Deployment is successfull"
OBOBOBOBOBOBOBOBOB                        }
                        else{
OBOB                            echo "Deployment is failed, performing rollback"
                            if(releaseExists.isEmpty()){
OB                                error "Deployment failed, not able to rollback, since it is first time deployment"
                            }
                            else{
                                sh """
                                aws eks update-kubeconfig --region ${region} --name ${project}-dev
                                helm rollback backend -n ${project} 0
                                sleep 60
                                """
                                rollbackStatus = sh(script: "kubectl rollout status deployment/backend -n expense --timeout=2m || true", returnStdout: true).trim()
                                if(rollbackStatus.contains('successfully rolled out')){
                                    error "Deployment is failed, Rollback is successfull"
                                }
                                else{
                                    error "Deployment is failed, Rollback is failed"
                                }
                            }
                        }
                    }
                }
            }
            
            /* stage('Nexus Artifact Upload'){
                steps{
                    script{
                        nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: "${nexusUrl}",
                            groupId: 'com.expense',
                            version: "${appVersion}",
                            repository: "backend",
                            credentialsId: 'nexus-auth',
                            artifacts: [
                                [artifactId: "backend" ,
                                classifier: '',
                                file: "backend-" + "${appVersion}" + '.zip',
                                type: 'zip']
                            ]
                        )
                    }
                }
            } */
        }
        post { 
            always { 
                echo 'I will always say Hello again!'
                deleteDir()
            }
            success { 
                echo 'I will run when pipeline is success'
            }
            failure { 
                echo 'I will run when pipeline is failure'
            }
        }
    }
}
