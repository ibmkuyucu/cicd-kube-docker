pipeline {
    agent any
	tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
    }
    environment {
        registry = "ikuyucu/vprofile"
        registryCredential = "dockerhub"
    }
    stages{

        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }
        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
        stage('CODE ANALYSIS with SONARQUBE') {
            environment {
                scannerHome = tool 'mysonarscanner4'
            }
            tools {
                jdk "OpenJDK11"
            }
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage("Build & Upload App Image") {
            steps {
                sh 'docker buildx build --platform linux/amd64,linux/arm64 -t $registry:V$BUILD_NUMBER -t $registry:latest --push .'
                // script {
                //     dockerImage = docker.build registry + ":V$BUILD_NUMBER"
                // }
            }
        }
        // stage("Upload Image") {
        //     steps {
        //         script {
        //             docker.withRegistry("", registryCredential) {
        //                 dockerImage.push("V$BUILD_NUMBER")
        //                 dockerImage.push("latest")
        //             }
        //         }
        //     }
        // }
        // stage("Remove Unused Images") {
        //     steps {
        //         sh "docker rmi $registry:V$BUILD_NUMBER"
        //     }
        // }
        stage("Kubernetes Deploy") {
            steps {
                sh "helm upgrade --install --force --namespace prod vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER}"
            }
        }
    }
}
