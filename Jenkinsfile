podTemplate(

yaml: '''
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: docker
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: "IfNotPresent"
    command:
    - cat
    tty: true
  - image: "jenkins/inbound-agent:4.10-3"
    name: "jnlp"
    resources:
      limits: {}
      requests:
        memory: "256Mi"
        cpu: "100m"
  restartPolicy: "Never"
''' ) {
    node(POD_LABEL) {
        stage('Checkout'){
            def scmVars = checkout([
                $class: 'GitSCM',
                branches: scm.branches,
                extensions: scm.extensions + [
                    [
                        $class: 'AuthorInChangelog'
                    ],
                    [
                        $class: 'ChangelogToBranch',
                        options: [
                            compareRemote: 'origin',
                            compareTarget: 'main'
                        ]
                    ]
                ],
                userRemoteConfigs: scm.userRemoteConfigs
            ])
        }
        stage ('Build') {
            container('kaniko') {
                script {
                    sh """
                        /kaniko/executor --context `pwd` --no-push
                    """
                }
            }
        }
    }
}