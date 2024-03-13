@Library('my_shared_library') _



pipeline {

    agent any

    parameters {
        string(name: "ImageName", defaultValue: "javapp", description: "name of the docker build")
        string(name: "ImageTag", defaultValue: "v1", description: "tag of the docker build")
        string(name: "DockerHubUser", defaultValue: "sarthakxxx03875", description: "name of docker user")
        string(name: "jfrogServer", defaultValue: "52.87.164.162", description: "IP of Jfrog server")
        choice(name: "action", choices: "create\ndestroy", description: "Choose create/Destroy")
    }

    stages {
        stage("Git Checkout"){
            when { expression {params.action == 'create'}}
            steps{
                gitCheckout (
                    branch : "main",
                    url: "https://github.com/sarthak-1996/test-app.git"
                )
            }
        }
        stage ("Unit Test : Maven") {
            when { expression {params.action == 'create'}}
            steps{
                script {
                    mvnTest()
                }
            }

        }
        stage ("Integration Test : Maven"){
            when {expression {params.action == 'create'}}
            steps{
                script {
                    integrationTest()
                }
            }
        }
        stage ("Code Analysis : SonarQube"){
            when {expression {params.action == 'create'}}
                steps{
                    script{
                        def SonarQubecredentialsId = 'sonarqube-api'
                        staticCodeAnalysis(SonarQubecredentialsId)
                    }
                }
        }
        stage("Quality Gates : SonarQube"){
            when {expression {params.action == 'create'}}
            steps{
                script{
                    def SonarQubecredentialsId = 'sonarqube-api'
                    qualityCheck(SonarQubecredentialsId)
                }
            }

        }
        stage ("Build : Maven"){
            when {expression {params.action == 'create'}}
            steps{
                script{
                    mvnBuild()
                }
            }
        }
        stage("Put Artifactory : JFrog"){
            when {expression {params.action == 'create'}}
            steps{
                script{
                    putArtifactory("${params.jfrogServer}")
                }
            }
        }
        stage("Build Image : Docker"){
            when {expression {params.action == 'create'}}
            steps{
                script{
                    dockerBuild("${params.DockerHubUser}", "${params.ImageName}")
                }
            }
        }
        stage("Scan Image : Trivy"){
            when {expression {params.action == 'create'}}
            steps{
                script{
                    imageScan("${params.DockerHubUser}", "${params.ImageName}")
                }
            }
        }
        stage("Push Image : DockerHub"){
            when {expression {params.action == 'create'}}
            steps{
                script{
                    pushImage("${params.DockerHubUser}", "${params.ImageName}")
                }
            }
        }
        stage("Image Cleanup: Docker"){
            when {expression {params.action == 'create'}}
            steps{
                script{
                    cleanImage("${params.DockerHubUser}", "${params.ImageName}")
                }
            }
        }
    }
    
}
