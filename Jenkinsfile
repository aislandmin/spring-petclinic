pipeline {
    agent any

    tools {
        maven 'maven39'
    }

    parameters {
        string(
            name: 'BRANCH_NAME',
            defaultValue: 'main'
        )
    }
    // Trigger every 5 minutes on Mondays
    // Format: MIN HOUR DOM MONTH DOW
    // H/5 = every 5 minutes, DOW=1 = Monday
    triggers {
        cron('H/5 * * * 1')
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                echo "Checking out source code from branch: ${params.BRANCH_NAME}"

                git branch: "${params.BRANCH_NAME}",
                    url: 'https://github.com/aislandmin/spring-petclinic'
            }
        }

        stage('Build') {
            steps {
                echo "Compiling source code..."
                bat 'mvn clean compile'
            }
        }

        stage('Unit Test') {
            steps {
                echo "Running unit tests..."
                bat 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Code Coverage (JaCoCo)') {
            steps {
                echo 'Generating JaCoCo coverage report...'
                bat 'mvn jacoco:report'
            }
        }

        stage('Publish Coverage') {
            steps {
                jacoco(
                    execPattern: 'target/jacoco.exec',
                    classPattern: 'target/classes',
                    sourcePattern: 'src/main/java'
                )
            }
        }

        stage('Package') {
            steps {
                echo "Packaging application..."
                bat 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Archive Artifacts') {
            steps {
                echo 'Archiving JAR and JaCoCo report...'
                // The build artifact (JAR)
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                // The JaCoCo HTML report
                archiveArtifacts artifacts: 'target/site/jacoco/**', fingerprint: true
            }
        }
    }

    post {
        always {
            echo 'Archiving JUnit test results...'
            junit 'target/surefire-reports/*.xml'
        }
    }
}
