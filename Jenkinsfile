#!/usr/bin/env groovy

@Library("release-pipeline") _

def appName = "cfssl"
def allServices = ["cfssl"]
def defaultBranch = "apatrynets/helm-chart"

KEYPR_IS_DEFAULT_BRANCH = env.BRANCH_NAME == defaultBranch

properties([
        disableConcurrentBuilds(), [$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false],
        parameters(
                [
                        booleanParam(
                                defaultValue: KEYPR_IS_DEFAULT_BRANCH,
                                description: 'Check to publish docker images',
                                name: 'KEYPR_PUBLISH_IMAGES'
                        ),
                        booleanParam(
                                defaultValue: KEYPR_IS_DEFAULT_BRANCH,
                                description: 'Check to deploy working branch to dev',
                                name: 'KEYPR_DEPLOY_TO_DEV'
                        ),

                ]),
        pipelineTriggers([])
])

node{

    KEYPR_PUBLISH_IMAGES = params.containsKey('KEYPR_PUBLISH_IMAGES') ? params.get('KEYPR_PUBLISH_IMAGES').toBoolean() : false
    KEYPR_DEPLOY_TO_DEV  = params.containsKey('KEYPR_DEPLOY_TO_DEV') ? params.KEYPR_DEPLOY_TO_DEV.toBoolean() : false

    stage('Checkout') {
        sh "env"

        // Clean the workspace before each build
        deleteDir()
        // Get code
        gitCheckout()

        // Get commit hash since it isn't exposed by default
        commitId = gitCommitId()
    }

    stage('Build images') {
        // Set image name
        imageId = "${appName}:${commitId}".trim()
        // Build the image once.
        appEnv = dockerBuild(appName, commitId)
        // Tag it for multiple deployments (we reuse the same image in all the deployments).
        for (dep in allServices) {
            if (dep != appName) {
                sh "docker tag $imageId $dep:$commitId"
            }
        }
    }

    stage('Publish & Deploy') {
        k8sDeploy(appName, [
                forceDevDeployment: KEYPR_DEPLOY_TO_DEV,
                forcePublish: KEYPR_PUBLISH_IMAGES,
                deployments: allServices
        ])
    }

    stage('Cleanup') {
        deleteDir()
    }
}