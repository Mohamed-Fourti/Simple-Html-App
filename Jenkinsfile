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
    
    stage('Build container in Portainer') {
        withEnv(['JWTTOKEN=${env.JWTTOKEN}']) {
            script {
                def gitRepo = 'https://github.com/Mohamed-Fourti/Simple-Html-App.git'
                def dockerfilePath = 'path/to/your/Dockerfile'
                def imageName = 'latest'
                def endpointURL = 'https://3.82.191.4:9443/api/endpoints/1/docker/build'

                def headers = [
                    'Content-Type': 'application/json',
                    'Authorization': "${JWTTOKEN}"
                ]
                
                def payload = [
                    'dockerfile': 'github.com/Mohamed-Fourti/Simple-Html-App.git',
                    'path': 'Dockerfile',
                    't': 'latest'
                ]

                def response = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', validResponseCodes: '200', httpMode: 'POST', ignoreSslErrors: true, consoleLogResponseBody: true, headers: headers, requestBody: groovy.json.JsonOutput.toJson(payload), url: endpointURL
                
                // Check response status and handle accordingly
                if (response.status == 200) {
                    echo "Container build successful in Portainer"
                } else {
                    error "Failed to build container in Portainer. Status code: ${response.status}"
                }
            }
        }
    }
}
