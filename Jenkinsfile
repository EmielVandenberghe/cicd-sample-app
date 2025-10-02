node {
    stage('Preparation') {
        // Stop en remove oude container
        catchError(buildResult: 'SUCCESS') {
            sh 'docker stop samplerunning'
            sh 'docker rm samplerunning'
        }
        // Cleanup workspace
        deleteDir()
    }
    
    stage('Checkout') {
        // Clone de repository
        git branch: 'main',
            url: 'https://github.com/USER/cicd-sample-app.git'
    }
    
    stage('Build') {
        // Build de Docker container
        sh 'bash ./sample-app.sh'
    }
    
    stage('Test') {
        // Get IPs dynamically
        script {
            def appIp = sh(
                script: "docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' samplerunning",
                returnStdout: true
            ).trim()
            
            def jenkinsIp = sh(
                script: "docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' jenkins_server",
                returnStdout: true
            ).trim()
            
            // Test de applicatie
            sh "curl http://${appIp}:5050/ | grep 'You are calling me from ${jenkinsIp}'"
        }
    }
    
    stage('Deploy') {
        echo 'Application deployed and running on http://192.168.56.20:5050/'
    }
}