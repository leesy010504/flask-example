pipeline {
  agent any
  tools {
    gradle 'my-gradle'
  }

  stages {
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
                  image: docker
                  command: [sleep]
                  args: [99d]
                  volumeMounts:
                    - name: docker-socket
                      mountPath: /var/run
                - name: docker-daemon
                  image: docker:dind
                  securityContext:
                    privileged: true
                  volumeMounts:
                    - name: docker-socket
                      mountPath: /var/run
                - name: kubectl
                  image: bitnami/kubectl:1.26.0
                  command: [sleep]
                  args: [99d]
                - name: gradle
                  image: gradle:7.5.1-jdk11
                  command: [sleep]
                  args: [99d]
          ''') {
            node(POD_LABEL) {
              container('docker') {
                checkout scm
              }
              container('gradle') {
                sh 'gradle build'
              }
            }
          }
        }
      }
    }
    stage('Build image') {
      steps {
        script {
          def app = docker.build("leesy010504/flask-example")
        }
      }
    }
    stage('Push image') {
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com/', 'dockerhub') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
          }
        }
      }
    }
  }
}
