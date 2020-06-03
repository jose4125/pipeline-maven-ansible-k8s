pipeline{
    agent any
    tools {
        maven "Maven3.6.3"
    }
    environment{
      PASS = credentials('docker-hub')
      USERHUB = "${env.USER_HUB}"
      COMMIT = "${env.BUILD_NUMBER}"
    }
    stages {
        stage("git checkout") {
            steps {
              git "https://github.com/yankils/hello-world"
            }
        }

        stage("Build"){
            steps{
                sh "mvn -version"
                sh "mvn clean install"
            }
        }
        stage("ansible container"){
            steps([$class: 'BapSshPromotionPublisherPlugin']) {
                sshPublisher(
                  continueOnError: false, failOnError: true,
                  publishers: [
                    sshPublisherDesc(
                      configName: "ansible-server",
                      verbose: true,
                      transfers: [
                        sshTransfer(
                          sourceFiles: "webapp/target/*.war",
                          remoteDirectory: "//opt//kubernetes",
                          removePrefix: "webapp/target",
                          execCommand: "ansible-playbook //opt/kubernetes/create-simple-devops-image.yml --extra-vars '{\"userFromJenkins\":$USERHUB,\"pass\":$PASS,\"imageTag\":$COMMIT}'"
                        )
                      ]
                    )
                  ]
                )
            }
            post{
                success{
                    echo "param commit value = $COMMIT"
                    build job: 'Deploy_on_kubernetes_using_ansible',
                    parameters: [
                      string(name: 'commit', value: "$COMMIT")
                    ]
                }
            }
        }
    }
}
