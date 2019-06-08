#!groovy

node {
    stage('Clone sources') {
        checkout scm;
    }

    def REPO_NAME = "paymentservice"
    def DOCKER_CONTEXT = "${WORKSPACE}"

    env.BRANCH_NAME = env.BRANCH_NAME ? env.BRANCH_NAME : 'master';
    def imageOwner = "dmitrybuhtiyarov"
    def deployTag = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
    def securityScanTag = "securityScan-${env.BUILD_NUMBER}"
    def imageName = "${imageOwner}/${REPO_NAME}"
    def registryFqdn = "registry.hub.docker.com"
    def registryUrl = "https://${registryFqdn}"
    def registryCredentialsId = "dockerhub_id"


    withCredentials([usernamePassword(credentialsId: 'dockerhub_id', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
        stage('Login') {
            sh "docker login -u ${USER} -p ${PASS} ${registryFqdn}"
        }

        stage('Build for security scan') {
            sh "docker build -t ${registryFqdn}/${imageName}:${securityScanTag} ${DOCKER_CONTEXT}"
        }

        stage('Push for security scan') {
            sh "docker push ${registryFqdn}/${imageName}:${securityScanTag}"
        }
    }

    docker.image('openjdk:8-jre-slim').inside("--entrypoint=''") {
        stage('Sonarqube') {
            withSonarQubeEnv('sonarqube') {
                def scannerHome = tool name: 'sonarqube'
                sh "${scannerHome}/bin/sonar-scanner"
            }
        }

        stage("Quality Gate") {
            timeout(time: 5, unit: 'MINUTES') {
                def qg = waitForQualityGate()
                if (qg.status != 'OK') {
                    error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
            }
        }
    }

    stage('Security scan') {
        sh "echo  '${registryFqdn}/${imageName}:${securityScanTag} ${DOCKER_CONTEXT}/Dockerfile' > anchore_images"
        anchore bailOnFail: false, forceAnalyze: true, name: 'anchore_images'
    }

    withCredentials([usernamePassword(credentialsId: 'dockerhub_id', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
        stage('Login') {
            sh "docker login -u ${USER} -p ${PASS} ${registryFqdn}"
        }

        stage('Push for security scan') {
            sh "docker tag ${registryFqdn}/${imageName}:${securityScanTag} ${registryFqdn}/${imageName}:${deployTag}"
            sh "docker push ${registryFqdn}/${imageName}:${deployTag}"
        }
    }

}
