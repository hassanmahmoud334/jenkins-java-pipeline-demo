pipeline {
    agent any

    environment {
        IMAGE_NAME = "hassanmahmoud334/java-jenkins-demo"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "📥 Checking out repository..."
                git branch: 'main', url: 'https://github.com/hassanmahmoud334/jenkins-java-pipeline-demo.git'
            }
        }

        stage('Build') {
            steps {
                echo "⚙️ Building Java app..."
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Check Build Number') {
            steps {
                script {
                    echo "🔢 Current Build Number: ${currentBuild.number}"
                    if (currentBuild.number > 5) {
                        error("❌ Build number (${currentBuild.number}) exceeded 5. Stopping pipeline!")
                    } else {
                        echo "✅ Build number (${currentBuild.number}) is OK."
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                echo "🐳 Building and pushing Docker image..."
                script {
                    sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."

                    withCredentials([string(credentialsId: 'dockerhub-pass', variable: 'DOCKERHUB_PASS')]) {
                        sh "echo $DOCKERHUB_PASS | docker login -u hassanmahmoud334 --password-stdin"
                    }

                    sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "🚀 Deploying container..."
                sh '''
                    docker stop java-demo || true
                    docker rm java-demo || true
                    docker run -d -p 9000:8080 --name java-demo ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
                echo "✅ Application deployed at http://localhost:9000"
            }
        }
    }

    post {
        success {
            echo "🎉 Build succeeded!"
        }
        failure {
            echo "❌ Build failed. Check logs for details."
        }
        always {
            echo "🧹 Cleaning up workspace..."
            cleanWs()
        }
    }
}
