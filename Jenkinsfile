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
    
    stage('Build image in Portainer') {
        withCredentials([usernamePassword(credentialsId: 'Portainer', usernameVariable: 'PORTAINER_USERNAME', passwordVariable: 'PORTAINER_PASSWORD')]) {
            script {
                def json = """
                    {
                        "imageName": "mohamedfourti/simple-app:latest",
                        "tag": "latest",
                        "repositoryURL": "https://github.com/mohamedfourti/simple-app.git",
                        "dockerfilePath": "Dockerfile"
                    }
                """
                def buildResponse = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', validResponseCodes: '200', httpMode: 'POST', ignoreSslErrors: true, consoleLogResponseBody: true, requestBody: json, httpHeader: [
                    Authorization: "${env.JWTTOKEN}"
                ], url: "https://3.82.191.4:9443/api/endpoints/1/docker/build"
                echo "Build response: ${buildResponse.status}"
                echo "Build output: ${buildResponse.content}"
            }
        }
    }
}
