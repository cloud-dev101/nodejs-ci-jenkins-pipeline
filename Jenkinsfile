pipeline {
    agent any
    tools {
        nodejs "node"
    }
    environment {
        GIT_SSH_COMMAND = 'ssh -o StrictHostKeyChecking=no' // Skip host key checking
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    // enter app directory, because that's where package.json is located
                    dir("app") {
                        // update application version in the package.json file with one of these release types: patch, minor or major
                        // this will commit the version update
                        sh "npm version patch"

                        // read the updated version from the package.json file
                        def packageJson = readJSON file: 'package.json'
                        def version = packageJson.version
                        def appName = packageJson.name

                        // set the new version as part of IMAGE_NAME
                        // env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                        env.IMAGE_NAME = "alchemistkay/$appName:$version"
                    }

                    // alternative solution without Pipeline Utility Steps plugin: 
                    // def version = sh (returnStdout: true, script: "grep 'version' package.json | cut -d '\"' -f4 | tr '\\n' '\\0'")
                    // env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
        stage('Run tests') {
            steps {
               script {
                    //enter app directory, because that's where package.json and tests are located
                    dir("app") {
                        // install all dependencies needed for running tests
                        sh "npm install"
                        sh "npm run test"
                    } 
               }
            }
        }
        stage('Build and Push docker image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'USER', passwordVariable: 'PASS')]){
                    sh "docker build -t ${IMAGE_NAME} ."
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh "docker push ${IMAGE_NAME}"
                }
            }
        }
        stage('commit version update') {
            steps {
                script {
                    sshagent (credentials: ['github-ssh-credential']) {
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'
                        sh 'git remote set-url origin git@github.com:alchemistkay/nodejs-ci-jenkins-pipeline.git'
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:main'
                    }
                }
            }
        }
    }
}


