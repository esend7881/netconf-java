// ----------------------------------------------------------------------------
//                                UNCLASSIFIED
//
//                          United States Government
//                      Department of the Navy (SSC Pacific)
//                              Unlimited Rights
// ----------------------------------------------------------------------------

pipeline {

    agent any

    options {
        timestamps()
        timeout(time: 1, unit: "HOURS")
    }

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-1.8-openjdk/"
        TMPDIR = "target"
        VERSION = readMavenPom().getVersion()
        ARTIFACT_ID = readMavenPom().getArtifactId()
        GROUP_ID = readMavenPom().getGroupId()
        SONAR_BRANCH = "${GIT_BRANCH}".replaceAll("/", "-")
    }

    tools {
        maven "Maven 3.6.3"
        jdk "OpenJDK 8 (Latest)"
    }

    stages {
        stage("Initialize") {
            steps {
                chuckNorris()
                echo sh(returnStdout: true, script: "env")
                notifyBitbucket0()
            }
        }

        stage("Build & Install") {
            steps {
                sh "mvn clean install"
            }
        }

//        stage("Sonarqube") {
//            steps{
//                script {
//                    withSonarQubeDI2E()
//                }
//            }
//        }
    }
    post {
        always {
            notifyBitbucket0()
        }
        success {
            script {
                nexusArtifactUploader0("Public_DI2E_Maven")
            }
        }
    }
}

def mavenVeryifyWithSonar(def host, def branch) {
    sh "mvn -P sonarqube sonar:sonar ${host} ${branch} -D sonar.projectKey=${GROUP_ID}:${ARTIFACT_ID}"
}

def withSonarQubeDI2E() {
    withSonarQubeEnv('Sonarqube Di2e Server') {
        script {
            def sonarScanner = tool name: 'SonarQube Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
            mavenVeryifyWithSonar("", "-D sonar.branch=${SONAR_BRANCH}")
        }
    }
}

def nexusArtifactUploader0(def repo) {
    nexusArtifactUploader artifacts: [
            [artifactId: "${ARTIFACT_ID}", file: "pom.xml", type: "pom"],
            [artifactId: "${ARTIFACT_ID}", file: "target/${ARTIFACT_ID}-${VERSION}.jar", type: "jar"],
            [artifactId: "${ARTIFACT_ID}", classifier: "sources", file: "target/${ARTIFACT_ID}-${VERSION}-sources.jar", type: "jar"],
            [artifactId: "${ARTIFACT_ID}", classifier: "javadoc", file: "target/${ARTIFACT_ID}-${VERSION}-javadoc.jar", type: "jar"]
    ],
            credentialsId: "687110ca-29bc-48c6-b35a-50b6040f1260",
            groupId: "${GROUP_ID}",
            nexusUrl: "nexus.di2e.net/nexus3",
            nexusVersion: "nexus3",
            protocol: "https",
            repository: "${repo}",
            version: "$VERSION"
}

def notifyBitbucket0() {
    notifyBitbucket commitSha1: "${GIT_COMMIT}",
            considerUnstableAsSuccess: false,
            credentialsId: "b901ff19-3484-442d-bc11-f6b3f2be7d26",
            disableInprogressNotification: false,
            ignoreUnverifiedSSLPeer: false,
            includeBuildNumberInKey: false,
            prependParentProjectKey: false,
            projectKey: "",
            stashServerBaseUrl: "https://bitbucket.di2e.net"
}