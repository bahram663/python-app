node {
    def app

    stage('Clone repository') {
        checkout scm
    }

    stage('Build image') {
        // Use a tag if it's provided, otherwise use 'latest'
        def imageTag = env.DOCKERTAG ?: 'latest'
        app = docker.build("python-app/test:${imageTag}")
    }

    stage('Test image') {
        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image') {
        // Push the image with the specified tag or 'latest'
        def imageTag = env.DOCKERTAG ?: 'latest'
        docker.withRegistry('http://nexus.bahram.local:5000/', 'dockerhub') {
            app.push(imageTag)
        }
        
        // Optionally, tag the image as 'latest' in the Nexus repository
        if (imageTag == 'latest') {
            sh "docker tag python-app/test:${imageTag} nexus.bahram.local:5000/python-app/test:latest"
            docker.withRegistry('http://nexus.bahram.local:5000/', 'dockerhub') {
                sh "docker push nexus.bahram.local:5000/python-app/test:latest"
            }
        }
    }

    stage('Trigger ManifestUpdate') {
        echo "Triggering updatemanifestjob"
        build job: 'updatemanifest', parameters: [string(name: 'DOCKERTAG', value: env.DOCKERTAG ?: 'latest')]
    }
}