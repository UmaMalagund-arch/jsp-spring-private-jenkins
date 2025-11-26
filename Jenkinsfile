pipeline {
    agent any

    tools {
        maven 'Maven-3.8'
        jdk 'JDK-17'
    }

    environment {
        GIT_URL = "https://github.com/your-private-repo/project.git"
        CREDS = "git-credentials-id"
        BUILD_JAR = ""
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: "${GIT_URL}",
                    credentialsId: "${CREDS}"
            }
            post {
                success { echo "Checkout successful" }
                failure { echo "Checkout failed" }
                always  { echo "Checkout stage completed" }
            }
        }

        stage('Maven Build') {
            steps {
                sh "mvn clean package -DskipTests"
            }
            post {
                success { echo "Build success" }
                failure { echo "Build failed" }
                always { echo "Maven Build stage completed" }
            }
        }

        stage('Detect JAR Name') {
            steps {
                script {
                    BUILD_JAR = sh(
                        script: "ls target/*.jar | head -n 1",
                        returnStdout: true
                    ).trim()

                    if (!BUILD_JAR) {
                        error("❌ No JAR file found in target/")
                    }

                    echo "Detected JAR: ${BUILD_JAR}"
                }
            }
            post {
                success { echo "Detected JAR file successfully" }
                failure { echo "Failed to detect JAR file" }
                always { echo "Jar detection stage completed" }
            }
        }

        stage('Run JAR with Conditions') {
            steps {
                script {

                    echo "Checking if application is already running..."

                    def pid = sh(
                        script: "pgrep -f ${BUILD_JAR} || true",
                        returnStdout: true
                    ).trim()

                    if (pid) {
                        echo "JAR running with PID ${pid}. Stopping..."
                        sh "kill -9 ${pid}"
                    } else {
                        echo "No running instance found."
                    }

                    // Remove old jar if present
                    sh """
                    if [ -f app.jar ]; then
                        echo "Removing old app.jar"
                        rm -f app.jar
                    fi
                    """

                    echo "Copying new JAR to app.jar"
                    sh "cp ${BUILD_JAR} app.jar"

                    echo "Starting application..."
                    sh "nohup java -jar app.jar > app.log 2>&1 &"
                }
            }
            post {
                success { echo "Application started successfully" }
                failure { echo "Error running the jar file" }
                always { echo "Run-JAR stage completed" }
            }
        }
    }

    post {
        success { echo "Pipeline completed successfully!" }
        failure { echo "Pipeline failed!" }
        always  { echo "Pipeline ended." }
    }
}
