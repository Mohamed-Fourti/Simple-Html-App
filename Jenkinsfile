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
        script {
            // Define the Portainer API URL and credentials
            def portainerUrl = 'https://3.82.191.4:9443/api'
            def portainerUsername = 'your_username'
            def portainerPassword = 'your_password'
            
            // Authenticate and retrieve the JWT token
            def authResponse = httpRequest(
                httpMode: 'POST',
                ignoreSslErrors: true,
                url: "${portainerUrl}/auth",
                contentType: 'APPLICATION_JSON',
                requestBody: [Username: portainerUsername, Password: portainerPassword]
            )
            def jwtToken = authResponse.getResponseHeaders().find { it.getName() == 'Authorization' }?.getValue()
            
            // Build the image using Portainer's API
            def buildResponse = httpRequest(
                httpMode: 'POST',
                ignoreSslErrors: true,
                url: "${portainerUrl}/endpoints/1/docker/build?t=test:latest&remote=https://github.com/your_username/Simple-Html-App&dockerfile=Dockerfile&nocache=true",
                contentType: 'APPLICATION_JSON',
                customHeaders: [[name: 'Authorization', value: jwtToken], [name: 'cache-control', value: 'no-cache']]
            )
            
            // Check the response status
            if (buildResponse.getResponseCode() == 200) {
                echo 'Docker image built successfully!'
            } else {
                error "Failed to build Docker image. Response: ${buildResponse.getResponseText()}"
            }
        }
    }


}
