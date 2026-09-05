pipeline {
    agent any
    tools {
        jdk 'JDK-21'
        maven 'Maven-3.9.9'
    }
    options {
        disableConcurrentBuilds()

        // Keep only the last 10 builds
        buildDiscarder(
            logRotator(
                numToKeepStr: '2'
            )
        )

        // Add timestamps to console output
        timestamps()

        // Maximum time for the entire pipeline
        timeout(time: 30, unit: 'MINUTES')
    }
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'sit', 'bat', 'nft', 'ite', 'prod'],
            description: 'Environment to deploy to'
        )
        choice(
            name: 'Deploy',
            choices: ['NO', 'YES'],
            description: 'Deploy after successful build?'
        )
        booleanParam(
            name: 'RUN_TESTS',
            defaultValue: true,
            description: 'Run unit tests'
        )
    }
    environment {
        APP_NAME = 'my-app'
        ENV = "${params.ENVIRONMENT}"
        DEP = "${params.Deploy}"
        TEST = "${params.RUN_TESTS}"        
    }    
    stages {
        stage('Cleanup') {
            steps {
                cleanWs()
            }
        }  
        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                dir('sample-git-repo') {
                    git branch: 'develop', url: 'https://github.com/nitins-devops/sample-git-repo.git'
                }
                dir('test-repo') {
                    git branch: 'main', url: 'https://github.com/nitins-devops/test-repo.git'
                }                
            }
        }
        stage('Test') {            
            steps {
                script {
                    echo "This is Test stage"
                    if( (ENV == "dev") && (TEST == "true") && (DEP == "YES") ){
                        echo "This is dev environment and want to execute test"
                    } else if ((ENV == "sit") && (TEST == "false") && (DEP == "NO") ) {
                        echo "This is sit environment and don't want to execute test"
                    } else {
                        echo "You have selected - ${ENV} and Test - ${TEST} - app name - ${APP_NAME}"
                        APP_NAME = "Updated value"                        
                    }
                }
            }
        }
        stage('store') {
            when {
                expression {
                    params.ENVIRONMENT != 'sit'
                }
            }
            steps {
                script {
                    echo "This is store stage"
                    echo "app name - ${APP_NAME}"
                    APP_NAME = "Updated again"
                }
            }
        }
        stage('Trigger CI-Job') {
            steps {
                script {
                    echo "Triggering downstream CI-Job..."

                    def downstreamBuild = build(
                        job: 'another-pipeline',
                        wait: true,
                        propagate: false,
                        parameters: [
                            string(
                                name: 'ENV',
                                value: params.ENVIRONMENT
                            ),
                            string(
                                name: 'DEP',
                                value: params.Deploy
                            ),
                            string(
                                name: 'TEST',
                                value: 'test'
                            )
                        ]
                    )

                    echo "======================================"
                    echo "Downstream Job: CI-Job"
                    echo "Build Number: ${downstreamBuild.number}"
                    echo "Build Result: ${downstreamBuild.result}"
                    echo "Build URL: ${downstreamBuild.absoluteUrl}"
                    echo "======================================"

                    if (downstreamBuild.result == 'SUCCESS') {
                        echo "another-pipeline completed successfully"

                    } else {
                        echo "another-pipeline failed with status: ${downstreamBuild.result}"
                        
                    }
                }
            }
        }        
        stage('modify') { 
            steps {
                echo "This is modify stage"
                echo "app name - ${APP_NAME}"
            }
        }        
    }
    post {
        always {
            echo "Build number: ${currentBuild.number}"
            echo "Build result: ${currentBuild.result}"
            echo "Build display name: ${currentBuild.displayName}"
            echo "Build URL: ${currentBuild.absoluteUrl}"
            echo "Build duration: ${currentBuild.durationString}"
        }
    }    
}
