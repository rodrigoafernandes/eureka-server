node {
    def REPO_GIT = "https://github.com/rodrigoafernandes/eureka-server.git"
    def BRANCH_NAME = "master"
    def PROJECT = "eureka-server"
    def mvnHome
    def TAG = sh(script: "env | grep gitlabSourceBranch | sed -e 's/[^0-9 .]//ig'", returnStdout: true).trim()

    stage("Clone project $PROJECT") {
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

    stage("Build $PROJECT") {
        if (TAG) {
            println ("Maven build on $PROJECT:$TAG")
        } else {
            println ("Maven build on $PROJECT")
        }
    }

    stage("Test $PROJECT") {
        println ("Maven Junit tests on $PROJECT")
    }

    stage('Quality Gates') {
        println ('Send code to analisys on SONAR')
    }

    stage('Push image to Registry') {
        println ("Build image of $PROJECT project and push to nexus private registry")
    }

    stage('Configure Centralized Environment Variables') {
        println ("Configure vault and consul variables for $PROJECT")
    }

    stage('Create K8s structures') {
        println ("Apply $PROJECT-namespace.yaml, $PROJECT-configmaps.yaml and $PROJECT-secrets.yaml")
    }

    stage('Create K8s deployment') {
        println ("Apply $PROJECT-deployment.yaml")
    }

    stage('Create K8s service') {
        println ("Apply $PROJECT-service.yaml")
    }

    stage('Verify health-check') {
        println ("Rollout K8s(sh(\"/bin/kubectl-110 --kubeconfig ~jenkins/kube/.kube/config rollout status -n $PROJECT deployment/$PROJECT-deployment\")), then check status")
    }
}