config = [:]
config.productVersion = "0.1"
config.buildVersion = "3.1"

try {
    // def project = utils.getRepoName()
    def coverageFilter = config.coverageFilter ?: []
    def noBuild = false
    def projects = ["AppEncryption", "Logging", "SecureMemory"]
    jenkinsutils.killOldBuilds()

    node('docker') {
        stage('checkout') {
            noBuild = buildUtils.checkoutGitHubRepo()
            if (noBuild) {
                return
            }
        }

        def version = version.getNewReleaseVersion(config.productVersion)

        try {
            def gdAspNetcoreImage = "gd-aspnetcore-build:${config.buildVersion ?: ASPNETCORE_BUILD_VERSION}"
            docker.withRegistry("https://${DOCKER_FLEXLINE_REPOSITORY}", 'effort_artifactory_user') {
                docker.image(gdAspNetcoreImage).inside {
                    def pwd = sh (script: "pwd", returnStdout: true)
                    for (project in projects)
                    {
                        sh "cd ${pwd}/csharp/${project}"
                        config.pattern = project

                        stage ('restore') {
                            buildUtils.dotnetRestore()
                        }
                        stage('test') {
                            buildUtils.dotnetTest(project, coverageFilter)
                        }
                        stage ('push') {
                            buildUtils.dotnetNuGetPushHelpers(config.pattern ?: 'Telephony', version)
                        }
                        stage ('release') {
                            githubutils.createRelease(
                                "v$version",
                                "#### [Artifactory](${NUGET_SMARTLINE_WEB_URL}$project/$version)\n"
                                + githubutils.getReleaseMessage())
                        }
                    }
                }
                githubutils.postStatus('success', version, 'Build')
                currentBuild.description = version
                currentBuild.result = 'SUCCESS'
            }
        } finally {
            step([$class: 'WsCleanup'])
        }
    }
    echo "Took ${utils.formatTime(System.currentTimeMillis() - currentBuild.startTimeInMillis)}"
}
finally {
    slackUtils.postOnMasterBuildFailure()
}
