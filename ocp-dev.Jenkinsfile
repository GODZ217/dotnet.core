node('maven') {
	def appName="test-netcore"
	def projectName="service-non-pnbp"
    
	def gitBranch="main"

    stage('Clone') {
            sh "git config --global http.sslVerify false"
            withCredentials([usernamePassword(credentialsId: 'gitlab-token', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            sh "git clone -b ${gitBranch} https://\${USERNAME}:\${PASSWORD}@git.atrbpn.go.id/suhayat.pusah/net-core-non-pnbp.git source "
        }
    }
    stage('App Build') {
        dir("source") {
            withCredentials([usernamePassword(credentialsId: 'bpn-nexus', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            sh "mvn clean deploy -Dmaven.test.skip=true -s settings.xml -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -Dmaven.wagon.http.ssl.ignore.validity.dates=true -Dnexus_username=${USERNAME}  -Dnexus_password=${PASSWORD}  "
            }
        }
    }
    stage('App Push'){
        dir("source"){
             sh "mkdir -p build-folder/target/ build-folder/app/"
            sh "cp ocp.Dockerfile build-folder/Dockerfile"

            sh "cp ./*.csproj build-folder/target/"
            
            def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim();
            
            sh "oc new-build --name=${appName} --image-stream=dotnet:7.0-ubi8 --binary=true -n ${projectName} || true"
            sh "oc start-build ${appName} --from-dir=build-folder/.  --follow --wait -n ${projectName} || true"
            sh "oc new-app ${appName} --name=${appName} -n ${projectName} || true"
            
            
            sh "oc tag cicd/${appName}:latest ${projectName}/${appName}:${commitHash} "
            sh "oc tag cicd/${appName}:latest ${projectName}/${appName}:latest "
        }
    }
    stage('App Deploy'){
        dir("source"){
            sh "sed 's,\\\$REGISTRY/\\\$HARBOR_NAMESPACE/\\\$APP_NAME:\\\$BUILD_NUMBER,image-registry.openshift-image-registry.svc:5000/${projectName}/${appName}:latest,g' ocp-kubernetes.yaml > kubernetes-ocp.yaml"
            sh "oc apply -f kubernetes-ocp.yaml -n ${projectName}"
            sh "oc set triggers deployment/${appName} --from-image=${projectName}/${appName}:latest -c ${appName} -n ${projectName} || true "
        }
    }
}

