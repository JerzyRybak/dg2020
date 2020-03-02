import groovy.json.JsonBuilder

node('jenkins-jenkins-slave') {
  withEnv(['REPOSITORY=dg2020']) {
    stage('Pull Image from Git') {
      script {
        git (url: "${scm.userRemoteConfigs[0].url}", credentialsId: "github-auth")
      }
    }
    stage('Build Image') {
      script {
        dbuild = docker.build("${REPOSITORY}:$BUILD_NUMBER")
      }
    }
    stage('Push Image to Registry') {
      script {
        docker.withRegistry("http://localhost:32000") {
          dbuild.push('latest')
        }
      }
    }
    stage('Deploy App to Kubernetes') {
      script {
        // secretNamespace: "default",
        // secretName: "cluster-registry2",
        kubernetesDeploy(configs: "app.yml",
                         kubeConfig: [path:"/kubeconfig"])
      }
    }
  }
}
