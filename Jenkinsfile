pipeline {
    agent {
        label 'maven'
    }

    parameters {
        booleanParam(name: 'CLEAN_BEFORE_BUILD', description: 'Limpar o workspace antes de rodar o build.' )
    }

    options {
        buildDiscarder(logRotator(daysToKeepStr: '7', numToKeepStr: '10'))
        disableConcurrentBuilds(abortPrevious: true)
          timeout(time: 10, unit: 'MINUTES')
    }

    stages {
        stage('Clean') {
            when {
                anyOf {
                    branch 'master'
                    expression { params.CLEAN_BEFORE_BUILD }
                }
            }
                        
            steps {
                mvn('clean')
            }
        }
        
        stage('Build') {
            steps {
                mvn('compile')
            }
        }

        stage('Test') {
            steps {
                mvn('verify')
                junit 'target/surefire-reports/*.xml'
            }
        }

        stage('Deploy') {
            when {
                branch 'master'
            }
            
            steps {
                script {
                    def artifactId = readPom('project.artifactId')
                    def version = readPom('project.version')

                    withCredentials([string(credentialsId: 'super-deploy-secret', variable: 'SUPER_CREDENTIALS')]) {
                        sh "./super-deploy.sh ${artifactId} ${version}"    // some block
                    }
                    
                    currentBuild.description = "Deploy completo do artefato ${artifactId} na versão ${version}"
                }
            }
        }

    }

}

def mvn(String args) {
    sh "mvn --no-transfer-progress -B $args"
}

def readPom(String property) {
    sh script: """mvn help:evaluate -Dexpression="${property}" -q -DforceStdout""", returnStdout: true
}