pipeline {
    agent any

    triggers {
        pollSCM('*/5 * * * 1-5')
    }

    options {
        skipDefaultCheckout(true)
        // Keep the 10 most recent builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }
    // Enviromental Variables Goes here
    environment {
      PATH="/var/lib/jenkins/miniconda3/bin:$PATH"
    }

    stages {

        stage ("Code pull"){
            steps{
                checkout scm
            }
        }

        stage('Build environment') {
            steps {
                echo "Building virtualenv and installing requirements"
                sh  '''virtualenv -p /usr/bin/python3 venv
                       source venv/bin/activate
                       pip3 install -r requirements/dev.txt
                    '''
            }
        }

        // stage('Static code metrics') {
        //     steps {
        //         echo "Install Dependences"
        //         sh  ''' source venv/bin/activate
        //                 python3 -m coverage xml -o reports/coverage.xml
        //             '''

        //         echo "Test coverage"
        //         sh  ''' source venv/bin/activate
        //                 coverage run irisvmpy/iris.py 1 1 2 3
        //                 python3 -m coverage xml -o reports/coverage.xml
        //             '''
        //         echo "Style check"
        //         sh  ''' source venv/bin/activate
        //                 pylint irisvmpy || true
        //             '''
        //     }
           
        // }



        stage('Unit tests') {
            steps {
                sh  ''' virtualenv -p /usr/bin/python3 venv
                        source venv/bin/activate
                        python3 -m pytest --verbose ./tests/test_dummy.py
                    '''
            }
            post {
                always {
                    // Archive unit tests for the future
                    junit allowEmptyResults: true, testResults: './tests/unit_tests.xml'
                }
            }
        }

        // stage('Acceptance tests') {
        //     steps {
        //         sh  ''' source venv/bin/activate
        //                 behave -f=formatters.cucumber_json:PrettyCucumberJSONFormatter -o ./reports/acceptance.json || true
        //             '''
        //     }
        //     post {
        //         always {
        //             pytest (buildStatus: 'SUCCESS',
        //             fileIncludePattern: '**/*.json',
        //             jsonReportDirectory: './reports/',
        //             parallelTesting: true,
        //             sortingMethod: 'ALPHABETICAL')
        //         }
        //     }
        // }

        stage('Deployment') {
            when {
                expression {
                    currentBuild.result == null || currentBuild.result == 'SUCCESS'
                }
            }
            steps {
                sh  '''
                        cd /var/lib/jenkins/workspace/Research-Website-Cloud-Infra/ansible/video-pipeline-deploy && ansible-playbook deploy.yaml  -i ../inventory 
                        cd /root/PraxiResearchCloud/terraform/GKE-CLUSTER-Template && terraform init --reconfigure
                        terraform validate
                    '''
            }
             
        }

 
    }

    post {
        always {
            sh 'echo post steps and virtual env cleanup goes here'
        }
        failure {
            emailext (
                subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                         <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']])
        }
    }
}