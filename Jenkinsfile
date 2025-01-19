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

                    image 'python:3.9'

                    args '-u root'

                }

            }

            steps {

                sh 'pip install pyinstaller'

                sh 'pyinstaller --onefile sources/add2vals.py'

                sleep time: 1, unit: 'MINUTES'

                echo 'Pipeline has finished successfully.'

            }

            post {

                success {

                    archiveArtifacts 'dist/add2vals'

                }

            }

        }
    }
}