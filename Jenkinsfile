def onmyduffynode(script){
    ansiColor('xterm'){
        timestamps{
            sh 'ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -l root ${DUFFY_NODE}.ci.centos.org -t \"export REPO=${REPO}; export BRANCH=${BRANCH};\" "' + script + '"'
        }
    }
}

def synctoduffynode(source)
{
    sh 'scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -r ' + source + " " + "root@" + "${DUFFY_NODE}.ci.centos.org:~/"
}

node('userspace-containerization'){

    stage('Checkout'){
        checkout scm
    }

    stage('Build'){
        try{
            stage ("Allocate node"){
                env.CICO_API_KEY = readFile("${env.HOME}/duffy.key").trim()
                duffy_rtn=sh(
                            script: "cico --debug node get --arch x86_64 -f value -c hostname -c comment",
                            returnStdout: true
                            ).trim().tokenize(' ')
                env.DUFFY_NODE=duffy_rtn[0]
                env.DUFFY_SSID=duffy_rtn[1]
            }

            stage ("Setup"){
                onmyduffynode "yum -y install epel-release"
                onmyduffynode "yum -y install git python36-pip"
                onmyduffynode "pip3.6 install tox pre-commit"
                synctoduffynode "./." // copy all source files
            }

            def tasks = [:]
            tasks["Tests"] = {
                stage ("Tests"){
                    onmyduffynode "tox -e py36"
                }
            }
            tasks["Linters"] = {
                stage ("Linters"){
                    onmyduffynode "pre-commit run --all-files"
                }
            }
            parallel tasks
        } catch (e) {
            currentBuild.result = "FAILURE"
            throw e
        } finally {
            stage("Cleanup"){
                sh 'cico node done ${DUFFY_SSID}'
            }
        }
    }
}
