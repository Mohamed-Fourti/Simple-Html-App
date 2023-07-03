node {
    def app

    stage('Clone repository') {
        checkout scm
    }

    stage('Build image') {
        app = docker.build("mohamedfourti/simple-app:latest")
    }

    stage('Push image') {
        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
            app.push("${env.BUILD_NUMBER}")
        }
    }

    stage('Get JWT Token') {
        withCredentials([usernamePassword(credentialsId: 'Portainer', usernameVariable: 'PORTAINER_USERNAME', passwordVariable: 'PORTAINER_PASSWORD')]) {
            script {
                def json = """
                    {"Username": "${PORTAINER_USERNAME}", "Password": "${PORTAINER_PASSWORD}"}
                """
                def jwtResponse = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', validResponseCodes: '200', httpMode: 'POST', ignoreSslErrors: true, consoleLogResponseBody: true, requestBody: json, url: "https://3.82.191.4:9443/api/auth"
                def jwtObject = new groovy.json.JsonSlurper().parseText(jwtResponse.getContent())
                env.JWTTOKEN = "Bearer ${jwtObject.jwt}"
            }
        }
        echo "${env.JWTTOKEN}"
    }

    stage('Build Docker Image on Portainer') {
        
            withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GITHUB_USERNAME', passwordVariable: 'GITHUB_PASSWORD')]) {
                def portainerEndpoint = 'https://3.82.191.4:9443'
                def portainerAPI = "${portainerEndpoint}/api/endpoints/1/docker/build"
                def repoURL = "https://github.com/${GITHUB_USERNAME}/Simple-Html-App"
                def dockerfilePath = "Dockerfile"
    
                def buildParams = [
                    't': 'test:latest',
                    'remote': repoURL,
                    'dockerfile': dockerfilePath,
                    'nocache': true
                ]
    
                def headers = [
                    'Authorization': env.JWTTOKEN,
                    'cache-control': 'no-cache'
                ]
    
                def imageResponse = httpRequest(
                    httpMode: 'POST',
                    ignoreSslErrors: true,
                    url: portainerAPI,
                    contentType: 'APPLICATION_FORM',
                    requestBody: buildParams,
                    headers: headers
                )
    
                echo "Response status: ${imageResponse.status}"
                echo "Response body: ${imageResponse.content}"
            
        }
    }
}
