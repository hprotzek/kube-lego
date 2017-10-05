// vim: et:ts=4:sw=4:ft=groovy
def jenkinsSlack(type){
    def jobInfo = "\n » ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|job>) (<${env.BUILD_URL}/console|console>)"
    if (type == 'start'){
        slackSend color: 'blue', message: "build started${jobInfo}"
    }
    if (type == 'finish'){
        def buildColor = currentBuild.result == null? "good": "warning"
        def buildStatus = currentBuild.result == null? "SUCCESS": currentBuild.result
        slackSend color: buildColor, message: "build finished - ${buildStatus}${jobInfo}"
    }
}

def gitTags(commit) {
    sh("git tag --contains ${commit} > GIT_TAGS")
    def gitTags = readFile('GIT_TAGS').trim()
    sh('rm -f GIT_TAGS')
    if (gitTags == '') {
        return []
    }
    return gitTags.tokenize('\n')
}

def gitCommit() {
    sh('git rev-parse HEAD > GIT_COMMIT')
    def gitCommit = readFile('GIT_COMMIT').trim()
    sh('rm -f GIT_COMMIT')
    return gitCommit
}

def gitMasterBranchCommit() {
    sh('git rev-parse origin/master > GIT_MASTER_COMMIT')
    def gitCommit = readFile('GIT_MASTER_COMMIT').trim()
    sh('rm -f GIT_MASTER_COMMIT')
    return gitCommit
}

def onMasterBranch(){
    return gitCommit() == gitMasterBranchCommit()
}

def imageTags(){
    def gitTags = gitTags(gitCommit())
    if (gitTags == []) {
        return ["canary"]
    } else {
        return gitTags + ["latest"]
    }
}

node('docker-enabled'){
        def imageName = 'gcr.io/springer-nature-sandbox/kube-lego'
        def imageTag = 'jenkins-build'

        jenkinsSlack('start')

        stage 'Checkout source code'
        checkout scm

        container('ubuntu') {
            stage 'Test kube-lego'
            sh "make docker_test"
            step([$class: 'JUnitResultArchiver', testResults: '_test/test*.xml'])

            stage 'Build kube-lego'
            sh "make docker_build"

            stage 'Build docker image'
            sh "docker build --build-arg VCS_REF=${gitCommit().take(8)} -t ${imageName}:${imageTag} ."

            stage 'Push docker image'
            sh "gcloud docker -- push ${imageName}:${imageTag}"
            jenkinsSlack('finish')
         }
}
