pipeline {
    agent { label "docker" }    //Run everything on an agent with the docker daemon
    environment {
        IMAGE = readMavenPom().getArtifactId()    //Use Pipeline Utility Steps
        VERSION = readMavenPom().getVersion()
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    reuseNode true    //reuse the workspace on the agent defined at top-level\
                    image 'maven:3.5.0-jdk-8'
                }
            }
            steps {
                sh 'mvn clean install'
                junit(allowEmptyResults: true, testResults: '**/target/surefire-reports/TEST-*.xml')
            }
        }
        stage('Quality Analysis') {
            environment {
                SONAR = credentials('sonar') //use ‘sonar’ credentials
            }
            parallel {     // run Sonar Scan and Integration tests in parallel
                stage ("Integration Test"} {
                    steps {
                        echo 'Run integration tests here...'
                    }
                }
                stage("Sonar Scan") {
                    steps {
                        sh "mvn sonar:sonar -Dsonar.login=$SONAR_PSW"
                    }
                }
            }
        }
        stage('Build and Publish Image') {
            when {
                branch 'master'    //only run these steps on the master branch
            }
            steps {
                sh """
                    docker build -t ${IMAGE} .
                    docker tag ${IMAGE} ${IMAGE}:${VERSION}
                    docker push ${IMAGE}:${VERSION}
                """
            }
        }
    }
    post {
        failure {    // notify users when the Pipeline fails
            mail(to: 'me@example.com', subject: "Failed Pipeline", body: "Something is wrong.")
        }
    }
}
