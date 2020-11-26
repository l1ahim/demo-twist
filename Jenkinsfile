import jenkins.model.*
import hudson.AbortException

pipeline {
    agent any
    environment {
        IMAGE_NAME = 'matt/node'
        TARGET_CONTAINER = "${IMAGE_NAME}"
        BUILD_TAG = "latest"
    }

    stages {
        stage('Build') {
            steps {
                echo "docker build -t $IMAGE_NAME:$BUILD_TAG ."
            }
        }
        stage('twistlock scan') {
            steps {
                sh 'docker pull nginx'
                script {
                    try {
                        currentBuild.result = 'SUCCESS'
                        echo 'Running tests...'
                        prismaCloudScanImage ca: '',
                            cert: '',
                            //policy: 'high',
                            //compliancePolicy: 'high',
                            containerized: false,
                            dockerAddress: 'unix:///var/run/docker.sock',
                            gracePeriodDays: 0,
                            ignoreImageBuildTime: true,
                            key: '',
                            logLevel: 'info',
                            requirePackageUpdate: false,
                            timeout: 10,
                            repository: "${TARGET_CONTAINER}",
                            image: "nginx"
                    }
                    //prismaCloudScanImage ca: '', cert: '', dockerAddress: 'unix:///var/run/docker.sock', image: 'nginx:latest', key: '', logLevel: 'info', podmanPath: '', project: '', resultsFile: 'prisma-cloud-scan-results.json', ignoreImageBuildTime:true
                    //sh "docker ps -a"
                    catch(hudson.AbortException e) {
                        echo "abort exception caught:\n ${e}"
                        currentBuild.result = 'UNSTABLE'
                    }
                    catch(e) {
                        echo "exception caught:\n ${e}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
            
        
        stage('Deploy') {
            steps {
                script {
                    if (currentBuild.result == 'SUCCESS')
                        echo 'Deploying....'
                    else
                        echo 'Deployment skipped, problem occurred, possibly Twistlock scan found CVEs or compliance issues!'
                }
            }
        }
    }
}
