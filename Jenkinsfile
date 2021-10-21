@Library('keptn-library@4.1')_
def keptn = new sh.keptn.Keptn()

node {
    properties([
        parameters([
         string(defaultValue: 'qgproject', description: 'Name of your Keptn Project for Quality Gate Feedback ', name: 'Project', trim: false), 
         string(defaultValue: 'qualitystage', description: 'Stage in your Keptn project used for for Quality Gate Feedback', name: 'Stage', trim: false), 
         string(defaultValue: 'evalservice', description: 'Servicename used to keep SLIs and SLOs', name: 'Service', trim: false),
         choice(choices: ['dynatrace', 'prometheus',''], description: 'Select which monitoring tool should be configured as SLI provider', name: 'Monitoring', trim: false),
         choice(choices: ['basic', 'perftest'], description: 'Decide which set of SLIs you want to evaluate. The sample comes with: basic and perftest', name: 'SLI'),
         string(defaultValue: '660', description: 'Start timestamp or number of seconds from Now()', name: 'StartTime', trim: false),
         string(defaultValue: '60', description: 'End timestamp or number of seconds from Now(). If empty defaults to Now()', name: 'EndTime', trim: false),
         string(defaultValue: '3', description: 'How many minutes to wait until Keptn is done? 0 to not wait', name: 'WaitForResult'),
        ])
    ])

    def commit_id


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

    stage('Deploy') {
        
    }

    stage('Initialize Keptn') {
        // Initialize the Keptn Project - ensures the Keptn Project is created with the passed shipyard
        keptn.keptnInit project:"${params.Project}", service:"${params.Service}", stage:"${params.Stage}", monitoring:"${monitoring}" , shipyard:'keptn/shipyard.yaml'

        // Upload quality gate files
        //keptn.keptnAddResources('keptn/dynatrace/dynatrace.conf.yaml','dynatrace/dynatrace.conf.yaml')
        //keptn.keptnAddResources('keptn/dynatrace/sli.yaml','dynatrace/sli.yaml')
        keptn.keptnAddResources('keptn/slo.yaml','slo.yaml')
    }
    stage('Trigger Quality Gate') {
        echo "Quality Gates ONLY: Just triggering an SLI/SLO-based evaluation for the passed timeframe"


        // ToDo: Label ${commit_id}
        
        // Trigger an evaluation
        def keptnContext = keptn.sendStartEvaluationEvent starttime:"${params.StartTime}", endtime:"${params.EndTime}" 
        String keptn_bridge = env.KEPTN_BRIDGE
        echo "Open Keptns Bridge: ${keptn_bridge}/trace/${keptnContext}"
    }
    stage('Wait for Result') {
        waitTime = 0
        if(params.WaitForResult?.isInteger()) {
            waitTime = params.WaitForResult.toInteger()
        }

        if(waitTime > 0) {
            echo "Waiting until Keptn is done and returns the results"
            def result = keptn.waitForEvaluationDoneEvent setBuildResult:true, waitTime:waitTime
            echo "${result}"
        } else {
            echo "Not waiting for results. Please check the Keptns bridge for the details!"
        }
    }
}