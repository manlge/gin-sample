pipeline {
    agent {
        kubernetes {
            label 'jenkins-pod'
            defaultContainer 'jnlp'
            yaml """
            apiVersion: v1
            kind: Pod
            metadata:
              labels:
                app: jenkins-slave-pod
              annotations:
                k8s.aliyun.com/eci-cpu: 2
                k8s.aliyun.com/eci-memory: 4Gi
            spec:
              containers:
              - name: golang
                image: golang:1.12
                command:
                - cat
                tty: true
            """
        }
    }

    stages {
        stage('Build') {
            steps {
                container('golang') {
                    sh """
                    go build -mod vendor -v
                    """
                }
            }
        }


        stage('Deploy other') {
            when {
              not { branch "master" }
            }
            steps {
                container("kubectl") {
                    withKubeConfig(
                        [
                            // you can replace `mo` to yours
                            credentialsId: 'test-env',
                            serverUrl: 'https://kubernetes.default.svc.cluster.local'
                        ]
                    ) {
                        sh '''
                        kubectl apply -f `pwd`/deploy.yaml -n test
                        kubectl wait --for=condition=Ready pod -l app=gin-sample --timeout=60s -n test
                        '''
                    }
                }
            }
        }

    }
}
