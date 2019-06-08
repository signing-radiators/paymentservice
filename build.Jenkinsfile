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


    def image;
    docker.withRegistry(registryUrl, registryCredentialsId) {
        stage('Build for security scan') {
            image = docker.build("${imageName}:${securityScanTag}", DOCKER_CONTEXT);
        }

        stage('Push for security scan') {
            image.push();
        }
    }

    stage('Security scan') {
        sh "echo  '${registryFqdn}/${imageName}:${securityScanTag} ${DOCKER_CONTEXT}/Dockerfile' > anchore_images"
        anchore bailOnFail: false, forceAnalyze: true, name: 'anchore_images'
    }

    docker.withRegistry(registryUrl, registryCredentialsId) {
        stage('Push') {
            image.push("${deployTag}");
        }
    }

}
