podTemplate(
    label: 'mypod', 
    inheritFrom: 'default',
    containers: [
        containerTemplate(
            name: 'golang', 
            image: 'golang:1.10-alpine',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'docker', 
            image: 'docker:18.02',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'helm', 
            image: 'ibmcom/k8s-helm:v2.6.0',
            ttyEnabled: true,
            command: 'cat'
        )
    ],
    volumes: [
        hostPathVolume(
            hostPath: '/var/run/docker.sock',
            mountPath: '/var/run/docker.sock'
        )
    ]
) {
    node('mypod') {
        def commitId
        stage ('Extract') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }
        stage ('Build') {
            container ('golang') {
                sh 'CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .'
            }
        }
        def repository
        stage ('Docker') {
            container ('docker') {
                // def registryIp = sh(script: 'getent hosts registry.kube-system | awk \'{ print $1 ; exit }\'', returnStdout: true).trim()      
                    sh "echo '${env.DOCKER_HUB_TOKEN}' | docker login --username ${env.DOCKER_HUB_USER} --password-stdin"
                    sh "docker build -t ${env.DOCKER_HUB_USER}/hello:${commitId} ."
                    sh "docker push ${env.DOCKER_HUB_USER}/hello:${commitId}"
                
            }
        }
        stage ('Deploy') {
            container ('helm') {
                sh "/helm init --client-only --skip-refresh"
                sh "/helm upgrade --install --wait --set image.repository=${env.DOCKER_HUB_USER}/hello,image.tag=${commitId} hello hello"
            }
        }
    }
}