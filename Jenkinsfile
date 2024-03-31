pipeline {
    agent any
    tools { 
        maven 'DHT_MVN' 
        jdk 'DHT_SENSE' 
    }
    stages {
        stage('Checkout') {
            steps {
                git(url: 'https://github.com/riyapatel25/maven-samples-A6', branch: 'master')
            }
        }
        stage('Setup Bisect') {
            steps {
                script {
                    // Ensure we're on the right branch and the repo is up to date
                    sh 'git checkout master'
                    sh 'git pull'

                    // Start bisect process
                    sh 'git bisect start'
                    sh "git bisect bad 26438de182f7a00147b5e53e9408a3c3745ca509"
                    sh "git bisect good 34d31973a0cc1f3d77cd5038fc9c01eeba7ec183"
                }
            }
        }
        stage('Run Bisect') {
            steps {
                script {
                    // Continues until the bisect process is complete
                    while (true) {
                        def result = sh(script: 'mvn clean test', returnStatus: true)
                        if (result == 0) {
                            sh 'git bisect good'
                        } else {
                            sh 'git bisect bad'
                        }

                              // Detect if git bisect is done by checking the output message
                      def bisectOutput = sh(script: "git bisect log", returnStdout: true).trim()
                      echo bisectOutput // Log output for diagnosis

                      // Break the loop if "git bisect log" contains the "is the first bad commit" message
                      if (bisectOutput.contains("first bad commit")) {
                          echo "Bisect completed."
                          break
                      }
                        
                        // Detect if git bisect is done by checking for the exit code
                        try {
                            sh 'git bisect log' // This line is just for logging purposes.
                        } catch (Exception e) {
                            // Break the loop if git bisect is finished.
                            if (e.getMessage().contains("exit code 0")) {
                                echo "Bisect completed."
                                break
                            }
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
