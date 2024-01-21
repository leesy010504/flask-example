pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Docker pod deploy') {
      steps {
        script {
          podTemplate(yaml: '''
            apiVersion: v1
            kind: Pod
            spec:
              serviceAccountName: jenkins-admin
              volumes:
                - name: docker-socket
                  emptyDir: {}
              containers:
                - name: docker
                  image: docker:19.03-dind
                  command: [sleep]
                  args: [99d]
                  volumeMounts:
                    - name: docker-socket
                      mountPath: /var/run
                - name: docker-daemon
                  image: docker:19.03-dind
                  securityContext:
                    privileged: true
                  volumeMounts:
                    - name: docker-socket
                      mountPath: /var/run
                - name: kubectl
                  image: bitnami/kubectl:1.26.0
                  command: [sleep]
                  args: [99d]
          ''') {
            node(POD_LABEL) {
              container('docker') {
                def imageName = "leesy010504/flask-test"
                def tag = "${env.BUILD_NUMBER}"
				withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                }
                sh 'docker build -t ${imageName}:${tag} .'
                sh 'docker push ${imageName}:${tag}'
              }
            }
          }
        }
      }
    }
  }
}
