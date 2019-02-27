pipeline {
  agent {
    kubernetes {
      //cloud 'kubernetes'
      label 'mypod'
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.6-jdk-8
    command: ['cat']
    tty: true
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: jenkins-docker-cfg
        mountPath: /kaniko/.docker
  volumes:
  - name: jenkins-docker-cfg
    projected:
      sources:
      - secret:
          name: devrepo-secret
          items:
            - key: .dockerconfigjson
              path: config.json
"""
    }
  }
  stages {
    stage('build maven') {
        steps {
            container('maven') {
                dir ('daytrader-webapp') {
                    sh 'mvn package -B -e -Dmaven.test.skip=true'
                    sh 'pwd'
                    sh 'ls -R | grep target'
                }
                sh 'pwd'
                sh 'ls -la'
                sh 'ls -la daytrader-webapp/daytrader-web/target'
            }
        }
    }
        stage('Build with Kaniko') {
      environment {
        PATH = "/busybox:/kaniko:$PATH"
      }
      steps {
        container(name: 'kaniko', shell: '/busybox/sh') {
            sh '''#!/busybox/sh
            /kaniko/executor -v debug -f `pwd`/daytrader-webapp/daytrader-web/Dockerfile -c `pwd`/daytrader-webapp/daytrader-web --insecure --skip-tls-verify --destination=baserepodev.devrepo.malibu-pctn.com/104017-malibu-artifacts/daytrader-example-webapp:latest \
            --build-arg WAR_ARTIFACTID=daytrader-web \
            --build-arg APP_VERSION=4.0.0 \
            --build-arg APP_ARTIFACTID=daytrader-webapp \
            --build-arg EXPOSE_PORT=5443
            '''
        }
      }
    }
  }
}
