pipeline {
    agent { label 'ec2orchestrator'}

    environment {
        //Input parameters
        param_git_branch = "${params.GitBranch}"
        param_profile = "${params.Profile}"
        param_legion_version = "${params.LegionVersion}"
        param_legion_infra_version = "${params.LegionInfraVersion}"
        param_enclave_name = "${params.EnclaveName}"
        param_docker_repo = "${params.DockerRepo}"
        param_debug_run = "${params.DebugRun}"
        //Job parameters
        sharedLibPath = "pipelines/legionPipeline.groovy"
        ansibleHome =  "/opt/legion/ansible"
        ansibleVerbose = '-v'
        cleanupContainerVersion = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                checkout scm
                script {
                    legion = load "${env.sharedLibPath}"
                    legion.buildDescription()
                }
            }
        }
        
        stage('Terminate Legion Enclave') {
            steps {
                script {
                    legion.ansibleDebugRunCheck(env.param_debug_run)
                    legion.authorizeJenkinsAgent()
                    print ("Remove Enclave")
                    legion.terminateLegionEnclave()
                }
            }
        }
    }
    
    post {
        always {
            script {
                legion = load "${sharedLibPath}"
                legion.notifyBuild(currentBuild.currentResult)
            }
        }
        cleanup {
            script {
                legion.cleanupClusterSg(param_legion_infra_version ?: cleanupContainerVersion)
            }
            deleteDir()
        }
    }
}