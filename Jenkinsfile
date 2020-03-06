#!/usr/bin/env groovy

final def pipelineSdkVersion = 'master'

pipeline {
    agent any
    options {
        timeout(time: 120, unit: 'MINUTES')
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
        skipDefaultCheckout()
    }
    stages {

	stage('Init') {
            steps {
                milestone 10
                library "s4sdk-pipeline-library@${pipelineSdkVersion}"
                stageInitS4sdkPipeline script: this
                abortOldBuilds script: this
            }
        }

	stage('Build and Test') {
            steps {
                milestone 20
                stageBuild script: this
            }
        }

        stage('Production Deployment') {
            when { expression { commonPipelineEnvironment.configuration.runStage.PRODUCTION_DEPLOYMENT } }
            //milestone 80 is set in stageProductionDeployment
            steps { stageProductionDeployment script: this }
        }

    }
    post {
        always {
            script {
                debugReportArchive script: this
                if (commonPipelineEnvironment?.configuration?.runStage?.SEND_NOTIFICATION) {
                    postActionSendNotification script: this
                }
                postActionCleanupStashesLocks script: this
                sendAnalytics script: this

                if (commonPipelineEnvironment?.configuration?.runStage?.POST_PIPELINE_HOOK) {
                    stage('Post Pipeline Hook') {
                        stagePostPipelineHook script: this
                    }
                }
            }
        }
        success {
            script {
                if (commonPipelineEnvironment?.configuration?.runStage?.ARCHIVE_REPORT) {
                    postActionArchiveReport script: this
                }
            }
        }
        failure {
            deleteDir()
        }
    }
}
