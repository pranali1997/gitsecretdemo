pipeline {
    agent {
        kubernetes {
            // Rather than inline YAML, in a multibranch Pipeline you could use: yamlFile 'jenkins-pod.yaml'
            // Or, to avoid YAML:
            // containerTemplate {
            //     name 'shell'
            //     image 'ubuntu'
            //     command 'sleep'
            //     args 'infinity'
            // }
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: ubuntu
    command:
    - sleep
    args:
    - infinity
  - name: helmcontainer
    image: projectwave/helm
    command:
    - sleep
    args:
    - infinity
  - name: gitsecretcontainer
    image: izhar0407/git-secrets
    command:
    - sleep
    args:
    - infinity
'''
            // Can also wrap individual steps:
            // container('shell') {
            //     sh 'hostname'
            // }
            defaultContainer 'shell'
        }
    }
    environment{
        PUBLIC_KEY=credentials('gpg-ownertrust')
        PRIVATE_KEY=credentials('gpg-secret')
        GPG_PASSPHRASE=credentials('gpg_passphrase')
        KUBECONFIG  = credentials('kubernetes-central')

    }
    stages {
        stage('Main') {
            steps {
                sh 'hostname'
            }
        }
        stage('helm') {
            steps {
                container('gitsecretcontainer'){
                  sh 'gpg --version'
                  sh 'gpg --batch --import ${PRIVATE_KEY}'
                  sh 'gpg --import-ownertrust ${PUBLIC_KEY}'
                  sh 'gpg --list-keys'
                  sh 'git secret reveal -p ${GPG_PASSPHRASE}'
                  sh 'git secret cat postgresqlhelmsecret/secrets.yaml'
                }
            }
        }
        stage('run') {
            steps {
                container('helmcontainer'){
                    sh 'helm version'
                    sh 'helm install postgresqldemo postgresqlhelmsecret --values postgresqlhelmsecret/secrets.yaml -n infra --kubeconfig=${KUBECONFIG}'
                }
            }
        }
    }
}
