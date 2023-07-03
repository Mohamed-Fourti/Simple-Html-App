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
      steps {
        script {
          // Build the image
          withCredentials([usernamePassword(credentialsId: 'Github', usernameVariable: 'GITHUB_USERNAME', passwordVariable: 'GITHUB_PASSWORD')]) {
              def repoURL = """
                https://3.82.191.4:9443/api/endpoints/1/docker/build?t=reactApp:latest&remote=https://$GITHUB_USERNAME:$GITHUB_PASSWORD@github.com/$GITHUB_USERNAME/Simple-Html-App.git&dockerfile=Dockerfile&nocache=true
              """
              def imageResponse = httpRequest httpMode: 'POST', ignoreSslErrors: true, url: repoURL, validResponseCodes: '200', customHeaders:[[name:"Authorization", value: env.JWTTOKEN ], [name: "cache-control", value: "no-cache"]]
          }
        }
      }
    }
}
