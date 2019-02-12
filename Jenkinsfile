node {

    def PROJECT = "eureka-server"
    def mvnHome

    stage('Clone project $PROJECT') {
        println ('Clone $PROJECT from SCM')
    }

    stage('Build $PROJECT') {
        println ('Maven build on $PROJECT')
    }

    stage('Test $PROJECT') {
        println ('Maven Junit tests on $PROJECT')
    }

    stage('Quality Gates') {
        println ('Send code to analisys on SONAR')
    }

    stage('Push image to Registry') {
        println ('Build image of $PROJECT project and push to nexus private registry')
    }

    stage('Configure Centralized Environment Variables') {
        println ('Configure vault and consul variables for $PROJECT')
    }

    stage('Create K8s structures') {
        println ('Apply $PROJECT-namespace.yaml, $PROJECT-configmaps.yaml and $PROJECT-secrets.yaml')
    }

    stage('Create K8s deployment') {
        println ('Apply $PROJECT-deployment.yaml')
    }

    stage('Create K8s service') {
        println ('Apply $PROJECT-service.yaml')
    }

    stage('Verify health-check') {
        println ('Rollout K8s(sh(\"/bin/kubectl-110 --kubeconfig ~jenkins/kube/.kube/config rollout status -n $PROJECT deployment/$PROJECT-deployment\")), then check status')
    }
}