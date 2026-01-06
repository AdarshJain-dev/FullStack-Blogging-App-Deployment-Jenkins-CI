pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        DOCKERHUB_REPO = "adarshjain428/blogging-app"
        CD_REPO = "https://github.com/AdarshJain-dev/FullStack-BloggingApp-Deployment-Argo-CD.git"
        IMAGE_TAG = "${BUILD_NUMBER}"
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {

        stage('Git Checkout - CI Repo') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/AdarshJain-dev/FullStack-Blogging-App-Deployment-Jenkins-CI.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile -DskipTests"
            }
        }

        stage('Test') {
            steps {
                sh "mvn test -DskipTests"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Multitier -Dsonar.projectName=Multitier -Dsonar.java.binaries=target"
                }
            }
        }
        stage('Package') {
            steps {
                sh "mvn package -DskipTests"
            }
        }
        // stage('Publish Artifacts to Nexus') {
        //     steps {
        //         withMaven(globalMavenSettingsConfig: 'maven-settings', maven: 'maven3', traceability: true) {
        //             sh "mvn deploy -DskipTests=true"
        //         }
        //     }
        // }
        stage('Docker build & Tag Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                        sh "docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} ."
                    }
                }
            }
        }
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                        sh "docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}"
                    }
                }
            }
        }
        stage('Update Image Tag in CD Repo') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'git-cred',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    sh """
                      rm -rf cd-repo
                      git clone https://${GIT_USER}:${GIT_PASS}@github.com/AdarshJain-dev/FullStack-BloggingApp-Deployment-Argo-CD.git cd-repo
                      cd cd-repo
        
                      sed -i 's|image: .*|image: ${DOCKERHUB_REPO}:${IMAGE_TAG}|g' deployment-service.yaml
        
                      git config user.email "jenkins@ci.com"
                      git config user.name "Jenkins CI"
        
                      git add deployment-service.yaml
                      git commit -m "Update image tag to ${IMAGE_TAG}"
                      git push origin main
                    """
                }
            }
        }
    }
    post {
        success {
            echo "CI completed. Image pushed and CD repo updated. Argo CD will sync automatically."
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}
