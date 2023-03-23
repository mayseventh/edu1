def label = "agent-${UUID.randomUUID().toString()}"
def gitBranch = 'master'
def docker_registry = "docker.io"  
def imageName = "ets0057/jenkinsci"

def TAG = getTag(gitBranch)

podTemplate(label: label, serviceAccount: 'jenkins-admin', namespace: 'edu11',
    containers: [
        containerTemplate(name: 'podman', image: 'mattermost/podman:1.8.0', ttyEnabled: true, command: 'cat', privileged: true, alwaysPullImage: true)
        ,containerTemplate(name: 'jnlp', image: 'jenkins/jnlp-slave:latest-jdk11', args: '${computer.jnlpmac} ${computer.name}')
    ],
    volumes: [
        hostPathVolume(hostPath: '/etc/containers' , mountPath: '/var/lib/containers' ),
        persistentVolumeClaim(mountPath: '/var/jenkins_home', claimName: 'jenkins-edu11-slave-pvc',readOnly: false)
        ]){    
    //node("podman-agent") {
    
    node(label) {
       stage('Clone Git Project') {
            sh "pwd"
            echo 'Clone'
            git branch: 'master', credentialsId: 'github_ci', url: 'https://github.com/mayseventh/edu1.git'
            sh "ls"
        }
                             // podman login -u ${USERNAME} -p ${PASSWORD} --log-level=debug ${docker_registry} --tls-verify=false

        stage('Build with Podman') {
                container('podman') {
                  withCredentials([usernamePassword(credentialsId: 'docker_ci',usernameVariable: 'USERNAME',passwordVariable: 'PASSWORD')]) {
                     sh  """
                     ls -al /etc/containers
                     podman login -u ${USERNAME} -p ${PASSWORD} --log-level=debug ${docker_registry} --tls-verify=false
                     podman build -t ${imageName} --cgroup-manager=cgroupfs --tls-verify=false .
                     podman push ${imageName} --tls-verify=false
                     echo 'TAG ==========> ' ${imageName}
                     """

                  }
              }
        }

    }

}

def getTag(branchName){     
    def TAG
    def DATETIME_TAG = new Date().format('yyyyMMddHHmmss')
    TAG = "${DATETIME_TAG}"
    return TAG
}  
