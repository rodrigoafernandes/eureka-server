node {

    def PROJECT = "eureka-server"
    try {
        def REPO_GIT = "https://github.com/rodrigoafernandes/eureka-server.git"
        def BRANCH_NAME = "master"
        def mvnHome = tool 'maven-jenkins'
        def sonarHome = tool 'sonarqube'
        def SONARQUBE_BIN = "$sonarHome/bin"
        def SONARQUBE_SERVER = "http://159.203.83.253:5555"
        def SONARQUBE_PKEY = "$PROJECT"
        def SONARQUBE_DASHBOARD = "$SONARQUBE_SERVER/dashboard?id=$SONARQUBE_PKEY"


        message = "PIPELINE STARTED - Build $BUILD_NUMBER"

        notifyBuild(message)

        stage('Clone project') {
            println(sh(script: "env | grep gitlabSourceBranch | sed -e 's/[^0-9 .]//ig'", returnStdout: true).trim())
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

        stage('Maven Build') {
            sh """
                export PATH=/usr/local/bin:/usr/bin:/bin:$mvnHome/bin
                mvn -P nexus clean package -DskipTests
            """

        }

        stage('Unit Tests') {
            sh """
                export PATH=/usr/local/bin:/usr/bin:/bin:$mvnHome/bin
                mvn test
            """
        }

        stage('Quality Gates') {
            sh("$SONARQUBE_BIN/sonar-scanner -Dsonar.host.url=$SONARQUBE_SERVER \
                -Dsonar.projectKey=$SONARQUBE_PKEY \
                -Dsonar.sources=$WORKSPACE/src \
                -Dsonar.login=b12fd5ba53edf35331d88f0579f501ed5d3f142d \
                -Dsonar.java.binaries=$WORKSPACE/target; sleep 3")
            sh(script: "curl -q -s -o $WORKSPACE/sonar_status.txt $SONARQUBE_SERVER/api/qualitygates/project_status?projectKey=$SONARQUBE_PKEY")

            def SONAR_STATUS = sh(script: "cat $WORKSPACE/sonar_status.txt | cut -c 29-30", returnStdout: true).trim()
            
            if ( SONAR_STATUS != "OK") {
                message = "THE CODE WAS NOT APPROVED BY SONARQUBE, GO CHECK -> $SONARQUBE_DASHBOARD"
                notifyBuild(message, "SONARQUBE")
            } else {
                notifyBuild("THE CODE WAS APPROVED BY SONARQUBE, GO CHECK -> $SONARQUBE_DASHBOARD", "INFO")
            }
        }

        stage('Push image to Registry') {
            sh """
                docker login -u ${env.DOCKER_REG_USR} -p ${env.DOCKER_REG_PWD} ${env.DOCKER_REG_PROXY}
                docker login -u ${env.DOCKER_REG_USR} -p ${env.DOCKER_REG_PWD} ${DOCKER_REG_PRIV}
                docker build -t ${env.DOCKER_REG_PRIV}/$PROJECT:${env.BUILD_NUMBER} -f docker/Dockerfile .
                docker push ${env.DOCKER_REG_PRIV}/$PROJECT:${env.BUILD_NUMBER}
            """
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
    def SLACK_TOKEN = "kAfbdUFVo3SEntGs71gHS3mF"
    def SLACK_CHANNEL = "#eureka-server"

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