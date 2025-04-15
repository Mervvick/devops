pipeline {
    agent { 
        docker { 
            image 'golang:1.22-alpine' 
            args '-v ${WORKSPACE}:/go/src/devops_pr --user root'
        } 
    }

    environment {
        GO111MODULE = 'on'
        GOCACHE = "/go/src/devops_pr/.cache/go-build"
    }

    triggers {
        githubPush()
    }
    
    stages {

        stage('Checkout') {
            steps {
                // Check code from SCM (Git) for all branches
                checkout([$class: 'GitSCM', 
                          branches: [[name: '**']], 
                          userRemoteConfigs: [[
                              url: 'git@github.com:JustNik8/devops_pr.git',
                              credentialsId: 'github-ssh-key'
                          ]]])
            }
        }

        stage('Check Go Environment') {
            steps {
                sh 'go version'
            }
        }

        stage('Prepare Environment') {
            steps {
                script {
                    // Create dir for cache if it doesn't exist
                    sh '''
                        mkdir -p /go/src/devops_pr/.cache/go-build
                        apk add --no-cache curl git
                        curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin v1.54.2
                        git config --global --add safe.directory .
                    '''
                }
            }
        }

        stage('Lint') {
            steps {
                script {
                    sh '''
                    golangci-lint run -v ./...
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    sh '''
                    go mod tidy
                    go build -o devops_pr -v ./cmd/web
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh 'go test ./...'
                }
            }
        }

    }
}
