//yesma mailey shared library use gareko diff repo mah sabai function haru define gareko cha so We follow DRY principle
// this is just CI and if this happens without error then only CD happens
@Library('Shared') _
pipeline {
    agent any
    
    environment{
        SONAR_HOME = tool "Sonar"
    }
    
    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
    }
    
    stages {
        stage("Validate Parameters") {
            steps {
                script {
                    if (params.FRONTEND_DOCKER_TAG == '' || params.BACKEND_DOCKER_TAG == '') {
                        error("FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                    }
                }
            }
        }
        stage("Workspace cleanup"){
            steps{
                script{
                    cleanWs() // Yo chai is an function of jenkins juust to cleanup the workspace halka space save garna ko lagi 
                }
            }
        }
        
        stage('Git: Code Checkout') {
            steps {
                script{
                    code_checkout("https://github.com/Avashneupane9857/DevSecOps-Pipeline/","master")
                }
            }
        }
        
        stage("Trivy: Filesystem scan"){
            steps{
                script{
                    trivy_scan()
                }
            }
        }

        stage("OWASP: Dependency check"){
            steps{
                script{
                    owasp_dependency()
                }
            }
        }
        
        stage("SonarQube: Code Analysis"){
            steps{
                script{
                    sonarqube_analysis("Sonar","DevSecOps","DevSecOps")
                }
            }
        }
        
        stage("SonarQube: Code Quality Gates"){
            steps{
                script{
                    sonarqube_code_quality()
                }
            }
        }
        
        stage('Exporting environment variables') {
            parallel{
                stage("Backend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatebackendnew.sh"
                            }
                        }
                    }
                }
                
                stage("Frontend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatefrontendnew.sh"
                            }
                        }
                    }
                }
            }
        }

       stage('Docker Check') {
  steps {
    sh 'whoami'
    sh 'docker version'
    sh 'docker info'
  }
}
 
        stage("Docker: Build Images"){
            steps{
                script{
                        dir('backend'){
                            docker_build("dev-sec-ops-backend-beta","${params.BACKEND_DOCKER_TAG}","avash9857")
                        }
                    
                        dir('frontend'){
                            docker_build("dev-sec-ops-frontend-beta","${params.FRONTEND_DOCKER_TAG}","avash9857")
                        }
                }
            }
        }
        
        stage("Docker: Push to DockerHub"){
            steps{
                script{
                    docker_push("dev-sec-ops-backend-beta","${params.BACKEND_DOCKER_TAG}","avash9857") 
                    docker_push("dev-sec-ops-frontend-beta","${params.FRONTEND_DOCKER_TAG}","avash9857")
                }
            }
        }
    }
    post{
        success{
            archiveArtifacts artifacts: '*.xml', followSymlinks: false
            build job: "DevSecOps-Pipeline-CD", parameters: [
                string(name: 'FRONTEND_DOCKER_TAG', value: "${params.FRONTEND_DOCKER_TAG}"),
                string(name: 'BACKEND_DOCKER_TAG', value: "${params.BACKEND_DOCKER_TAG}")
            ]
        }
    }
}
