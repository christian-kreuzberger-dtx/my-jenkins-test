@Library('keptn-library@5.1')_
def keptn = new sh.keptn.Keptn()


node('jenkins-slave') {

    def commit_id

    def image_name = "christiankreuzbergerdtx/docker-nodejs-demo"

    def project = "nodejs-example"
    def service = "hello-service"
    def firststage = "dev"

    stage('Preparation') {
        checkout scm
        sh "git rev-parse --short HEAD > .git/commit-id"                        
        commit_id = readFile('.git/commit-id').trim()
    }

    stage('Test') {
        nodejs(nodeJSInstallationName: 'nodejs') {
        sh 'npm install --only=dev'
        sh 'npm test'
        }
    }

    stage('Docker build/push') {
      docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
        def app = docker.build("${image_name}:${commit_id}", '.').push()
      }
    }

    stage('Initialize Keptn Project') {
        // Initialize the Keptn Project - ensures the Keptn Project is created with the passed shipyard
        keptn.keptnInit project:"${project}", service:"${service}", stage:"${firststage}", monitoring: "dynatrace", shipyard: ".keptn/shipyard.yaml"
    }

    stage('Add files to Keptn Project') {
        // Upload quality gate files
        //keptn.keptnAddResources('keptn/dynatrace/dynatrace.conf.yaml','dynatrace/dynatrace.conf.yaml')
        // Upload SLI and SLO files
        keptn.keptnAddResources('.keptn/dynatrace/sli.yaml','dynatrace/sli.yaml')
        keptn.keptnAddResources('.keptn/slo.yaml','slo.yaml')

        // Package Helm Chart .tgz
        sh 'tar -cvzf .keptn/helm/hello-service.tgz -C .keptn/helm/ hello-service'

        // Add Helm Chart for hello-service
        keptn.keptnAddResources('.keptn/helm/hello-service.tgz', 'helm/hello-service.tgz')
        // ToDo: Add helm chart for other stages too
        // Depends on https://github.com/keptn-sandbox/keptn-jenkins-library/issues/60
    }

    stage('Trigger Delivery') {
        def keptnContext = keptn.sendDeliveryTriggeredEvent image:"${image_name}:${commit_id}"
        String keptn_bridge = env.KEPTN_BRIDGE
        echo "Open Keptns Bridge: ${keptn_bridge}/trace/${keptnContext}"
    }

    stage('Wait for Result') {
        echo "Waiting until Keptn is done and returns the results"
        def result = keptn.waitForEvaluationDoneEvent setBuildResult:true, waitTime:5
        echo "${result}"
    }
}