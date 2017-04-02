podTemplate(label: 'mavenPod', inheritFrom: 'mypod',
        containers: [
                containerTemplate(name: 'maven', image: 'maven:3.3.9-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
        ],
        volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
) {

    node('mavenPod') {

        checkout scm

        def namespace = "click-count-${env.BRANCH_NAME}"

        def version
        sh 'env'
        sh "git rev-parse --short HEAD > GIT_COMMIT"
        version = readFile('GIT_COMMIT').take(6)
        String imageTag = "10.233.57.46:5000/xebiafrance/uo-click-count:${version}"

        stage('Build') {
            container('maven') {
                sh 'mvn -q -B clean package'
            }
        }

        stage('Results') {
            archive 'target/clickCount.war'
        }

        stage('Build image') {
            sh "docker build -t ${imageTag} ."
        }

        stage('Push image') {
            sh "docker push ${imageTag}"
        }

        stage('Deploy on Staging') {
            sh("kubectl get ns ${namespace} || kubectl create ns ${namespace}")
            sh "sed -i.bak 's#__FRONTEND_IMAGE__#${imageTag}#' ./k8s/dev/*.yml"
            sh "kubectl --namespace=${namespace} apply -f k8s/dev/"
            echo 'To access your environment run `kubectl proxy`'
            echo "Then access your service via http://localhost:8001/api/v1/proxy/namespaces/${namespace}/services/clickcount-service:8080/"
        }

    }
}