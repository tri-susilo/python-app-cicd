pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }

        stage('Manual Approval') {
            steps {
                script {
                    def userInput = input(
                        message: 'Lanjutkan ke tahap Deploy?',
                        parameters: [choice(choices: ['Yes'], description: 'Choose an option', name: 'CONTINUE')]
                    )
                    if (userInput == 'Yes') {
                        echo 'Continuing with deployment...'
                    } else {
                        echo 'Deployment aborted.'
                        sh './jenkins/kill.sh'
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'cdrx/pyinstaller-linux:python2'
                }
            }
            steps {
                sh 'pyinstaller --onefile sources/add2vals.py'
                sh 'sleep 60'
                sh './jenkins/kill.sh'
            }
        }
    }
}