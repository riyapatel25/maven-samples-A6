pipeline {
    agent any
    maven 'DHT_MVN' 
    jdk 'DHT_SENSE' 
    environment {
        // Set the known good and bad commits
        GOOD_COMMIT = '34d31973a0cc1f3d77cd5038fc9c01eeba7ec183'
        BAD_COMMIT = '26438de182f7a00147b5e53e9408a3c3745ca509'
    }
    stages {
        stage('Setup Bisect') {
            steps {
                script {
                    // Ensure we're on the right branch and the repo is up to date
                    sh 'git checkout master'
                    sh 'git pull'
                    
                    // Start bisect process
                    sh 'git bisect start'
                    sh "git bisect bad ${BAD_COMMIT}"
                    sh "git bisect good ${GOOD_COMMIT}"
                }
            }
        }
        stage('Run Bisect') {
            steps {
                script {
                    // Continues until the bisect process is complete
                    def bisectResult = ''
                    while (true) {
                        try {
                            // Try to run Maven clean test
                            sh 'mvn clean test'
                            bisectResult = 'good'
                        } catch (Exception e) {
                            bisectResult = 'bad'
                        }
                        
                        // Based on the result, mark the commit as good or bad
                        if (bisectResult == 'good') {
                            sh 'git bisect good'
                        } else {
                            sh 'git bisect bad'
                        }
                        
                        // Check if the bisect process has found the first bad commit
                        def bisectStatus = sh(script: "git bisect status", returnStdout: true).trim()
                        if (bisectStatus.contains("is the first bad commit")) {
                            echo bisectStatus
                            break
                        }
                    }
                }
            }
        }
        stage('Cleanup') {
            steps {
                // End the bisect process and clean up the environment
                sh 'git bisect reset'
            }
        }
    }
    post {
        always {
            // Clean up workspace after the pipeline is done
            cleanWs()
        }
    }
}
