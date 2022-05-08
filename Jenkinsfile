podTemplate(

yaml: '''
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: "IfNotPresent"
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: "/kaniko/.docker"
      name: "volume-1"
      readOnly: false
  - image: "jenkins/inbound-agent:4.10-3"
    name: "jnlp"
    resources:
      limits: {}
      requests:
        memory: "256Mi"
        cpu: "100m"
  restartPolicy: "Never"
  volumes:
  - configMap:
      name: "docker-config"
    name: "volume-1"
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
        stage ('Build FE') {
            container('kaniko') {
                script {
                    sh """
                        /kaniko/executor --context frontend/ --destination ariretiarno/bp-cilsy-14:frontend-${BUILD_NUMBER}
                    """
                }
            }
        }

        stage ('Build BE') {
            container('kaniko') {
                script {
                    sh """
                        /kaniko/executor --context backend/ --destination ariretiarno/bp-cilsy-14:backend-${BUILD_NUMBER}
                    """
                }
            }
        }
    }
}