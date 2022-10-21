pipeline {
    agent {
        label 'maven'
    }

    parameters {
        booleanParam(name: 'CLEAN_BEFORE_BUILD', description: 'Limpar o workspace antes de rodar o build.' )
    }

    stages {
        stage('Clean') {
            when {
                expression { params.CLEAN_BEFORE_BUILD }
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
            steps {
                script {
                    def artifactId = readPom('project.artifactId')
                    def version = readPom('project.version')

                    sh "./deploy.sh ${artifactId} ${version}"
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