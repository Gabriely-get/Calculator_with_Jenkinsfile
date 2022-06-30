pipeline {

    agent any

    stages {

        stage('Connect to Git Repository') {
            steps {
                git branch: 'main', url: "https://github.com/Gabriely-get/Calculator.git"
            }
        }

        stage('Gradle clean') {
            steps {
                sh './gradlew clean'
            }
        }

        stage('Gradle build') {
            steps {
                sh './gradlew build'
            }
        }

        stage('Upload Artifact in JFrog') {
            steps {
                rtUpload (
                    serverId: "Calculator Artifactory",
                    spec: """{
                                "files": [
                                    {
                                        "pattern": "build/libs/Calculator-1.0-all.jar",
                                        "target": "generic-calculator-build"
                                    }
                                ]
                    }"""
                )
            }
        }
    }
}
