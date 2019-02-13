node {
    try {
        def REPO_GIT = "https://github.com/rodrigoafernandes/eureka-server.git"
        def BRANCH_NAME = "master"
        def PROJECT = "eureka-server"
        def mvnHome = tool 'maven-jenkins';

        message = "PIPELINE STARTED - Build $BUILD_NUMBER"

        notifyBuild(message)

        stage("Clone project") {
            checkout([$class: 'GitSCM',
                        userRemoteConfigs: [[url: "$REPO_GIT", credentialsId: '5d0b7fd5-abfa-4738-a181-c89cd6d91599']],
                        branches: [[name: "$BRANCH_NAME"]],
                        clean: false,
                        extensions: [[$class: 'SubmoduleOption',
                                        disableSubmodules: false,
                                        parentCredentials: false,
                                        recursiveSubmodules: true,
                                        reference: '',
                                        trackingSubmodules: false]],
                        doGenerateSubmoduleConfigurations: false,
                        submoduleCfg: []
                    ]
            )
        }

        stage("Maven Build") {
            sh """
                export PATH=/usr/local/bin:/usr/bin:/bin:$mvnHome/bin
                mvn -P nexus clean package -DskipTests
            """

        }

        stage("Unit Tests") {
            sh """
                export PATH=/usr/local/bin:/usr/bin:/bin:$mvnHome/bin
                mvn test
            """
        }

        stage('Quality Gates') {
            message = "Send Code To Sonarqube"
            notifyBuild(message, "SONARQUBE")
        }

        stage('Push image to Registry') {
            println ("Build image of $PROJECT project and push to nexus private registry")
        }

        stage('Configure Centralized Environment Variables') {
            notifyBuild("Configure vault and consul variables for $PROJECT", "INFO")
        }

        stage('Create K8s structures') {
            println ("Apply $PROJECT-namespace.yaml, $PROJECT-configmaps.yaml and $PROJECT-secrets.yaml")
        }

        stage('Create K8s deployment') {
            println ("Apply $PROJECT-deployment.yaml")
        }

        stage('Create K8s service') {
            notifyBuild("Apply $PROJECT-service.yaml", "INFO")
        }

        stage('Verify health-check') {
            println ("Rollout K8s(sh(\"/bin/kubectl-110 --kubeconfig ~jenkins/kube/.kube/config rollout status -n $PROJECT deployment/$PROJECT-deployment\")), then check status")
            notifyBuild("", currentBuild.result)
        }
    } catch (error) {
        currentBuild.result = "FAILED"
        throw error
    } finally {
        notifyBuild("", currentBuild.result)
    }

}

def notifyBuild(String message, String buildStatus = 'STARTED') {
    def SLACK_URL = "https://gigiodevsoftware.slack.com/services/hooks/jenkins-ci/"
    def SLACK_TOKEN = "S8HLM5gtOx64tkN0eqCi7X3K"
    def SLACK_CHANNEL = "#jenkins"

    buildStatus = buildStatus ?: 'SUCCESSFUL'
    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.JOB_URL})"

    if (buildStatus == 'STARTED') {
        colorCode = '#FFFF00' // Yellow
    } else if (buildStatus == 'SUCCESSFUL' || buildStatus == 'SUCCESS') {
        colorCode = '#00FF00' // Green
    } else if (buildStatus == 'INFO') {
        colorCode = '#0080FF' // Blue
    } else {
        colorCode = '#FF0000' // Red
    }

    slackSend (color:"${colorCode}",
        baseUrl: SLACK_URL,
        token: SLACK_TOKEN,
        channel: SLACK_CHANNEL,
        message: "${summary} ${message}")
}