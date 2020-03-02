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
    parallel (
      "Test": {
        script {
          sh "while :; do sleep 1; done"
        }
        echo 'All functional tests passed'
      },
      "Check Image (pre-Registry)": {
        smartcheckScan([
          imageName: "${REPOSITORY}:$BUILD_NUMBER",
          smartcheckHost: "${DSSC_SERVICE}",
          smartcheckCredentialsId: "smartcheck-auth",
          insecureSkipTLSVerify: true,
          insecureSkipRegistryTLSVerify: true,
          preregistryScan: true,
          preregistryHost: "${DSSC_REGISTRY}",
          preregistryCredentialsId: "preregistry-auth",
          findingsThreshold: new groovy.json.JsonBuilder([
            malware: 0,
            vulnerabilities: [
              defcon1: 1000,
              critical: 1000,
              high: 1000,
            ],
            contents: [
              defcon1: 0,
              critical: 0,
              high: 0,
            ],
            checklists: [
              defcon1: 0,
              critical: 0,
              high: 0,
            ],
          ]).toString(),
        ])
      }
    )
    stage('Do something...') {
     withCredentials([usernamePassword(credentialsId: 'smartcheck-auth', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) { 
       sh "echo $USERNAME; echo $PASSWORD" 
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
                         kubeconfig: [path:"/kubeconfig"])
      }
    }
  }
}
