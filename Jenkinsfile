node {
    def server = Artifactory.newServer url: 'http://192.168.56.10:8081/artifactory', credentialsId: 'ARTIFACTORY_JENKINS_API_KEY' 
    def rtGradle = Artifactory.newGradleBuild()
    def buildInfo = Artifactory.newBuildInfo()

    stage ('Clone') {
        git url: 'https://github.com/jfrogdev/project-examples.git'
    }

    stage ('Artifactory configuration') {
        rtGradle.tool = GRADLE_TOOL // Tool name from Jenkins configuration
        rtGradle.deployer repo:'libs-snapshot-local',  server: server
        rtGradle.resolver repo:'jcenter', server: server
    }

    withEnv(['DONT_COLLECT=FOO']) {
        stage ('Config Build Info') {
            buildInfo.env.capture = true
            buildInfo.env.filter.addInclude("*")
            buildInfo.env.filter.addExclude("DONT_COLLECT*")
        }

        stage ('Extra gradle configurations') {
            rtGradle.deployer.artifactDeploymentPatterns.addExclude("*.war")
            rtGradle.usesPlugin = true // Artifactory plugin already defined in build script
        }

        stage ('Exec Gradle') {
            rtGradle.run rootDir: "gradle-examples/gradle-example/", buildFile: 'build.gradle', tasks: 'clean artifactoryPublish', buildInfo: buildInfo
        }

        stage ('Publish build info') {
            server.publishBuildInfo buildInfo
        }
    }
}
