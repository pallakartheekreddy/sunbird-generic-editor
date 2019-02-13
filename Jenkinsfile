node('build-slave') {
    try {
        String ANSI_GREEN = "\u001B[32m"
        String ANSI_NORMAL = "\u001B[0m"
        String ANSI_BOLD = "\u001B[1m"
        String ANSI_RED = "\u001B[31m"
        String ANSI_YELLOW = "\u001B[33m"
        
        ansiColor('xterm') {
            stage('Checkout') {
                cleanWs()
                def scmVars = checkout scm
                if(params.github_release_tag == ""){
                    checkout scm: [$class: 'GitSCM', branches: [[name: scmVars.GIT_BRANCH]], extensions: [[$class: 'CloneOption', depth: 5, noTags: false, reference: '', shallow: true], [$class: 'SubmoduleOption', disableSubmodules: false, parentCredentials: true, recursiveSubmodules: true, reference: '', trackingSubmodules: false]], userRemoteConfigs: [[url: scmVars.GIT_URL]]]
                    commit_hash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    branch_name = sh(script: 'git name-rev --name-only HEAD | rev | cut -d "/" -f1| rev', returnStdout: true).trim()
                    artifact_version = branch_name + "_" + commit_hash
                    println(ANSI_BOLD + ANSI_YELLOW + "github_release_tag not specified, using the latest commit hash: " + commit_hash + ANSI_NORMAL)
                }
                else {
                    checkout scm: [$class: 'GitSCM', branches: [[name: "refs/tags/$params.github_release_tag"]], extensions: [[$class: 'CloneOption', depth: 5, noTags: false, reference: '', shallow: true], [$class: 'SubmoduleOption', disableSubmodules: false, parentCredentials: true, recursiveSubmodules: true, reference: '', trackingSubmodules: false]], userRemoteConfigs: [[url: scmVars.GIT_URL]]]
                    artifact_version = params.tag
                    println(ANSI_BOLD + ANSI_YELLOW + "github_release_tag specified, building from github_release_tag: " + params.tag + ANSI_NORMAL)
                }
                echo "artifact_version: "+ artifact_version
            }
        }

        stage('Build') {
            sh """
                        rm -rf generic-editor
                        node -v
                        npm -v
                        
                        npm install
                        cd app
                        bower cache clean
                        bower install --force
                        cd ..
                        
                        version_number = 
                        build_number = 
                        
                        gulp packageCorePlugins
                        npm run plugin-build
                        npm run build
                        npm run test
                        
                 """    
        }

        stage('Archive artifacts'){
            sh """
                        mkdir generic_editor_artifacts
                        cp generic-editor.zip generic_editor_artifacts
                        zip -j generic_editor_artifacts.zip:${artifact_version} generic_editor_artifacts/*
                    """
            archiveArtifacts artifacts: "generic_editor_artifacts.zip:${artifact_version}", fingerprint: true, onlyIfSuccessful: true
            sh """echo {\\"artifact_name\\" : \\"lp_yarn_artifacts.zip\\", \\"artifact_version\\" : \\"${artifact_version}\\", \\"node_name\\" : \\"${env.NODE_NAME}\\"} > metadata.json"""
            archiveArtifacts artifacts: 'metadata.json', onlyIfSuccessful: true
            currentBuild.description = "${artifact_version}"
        }
    }

    catch (err) {
        currentBuild.result = "FAILURE"
        throw err
    }

}
