def CLOUD_ENV_MAP = [
    "dev": [
        "AccountID": "11111",
        "AssumeRole": "app-dev-role",
        "folderpath": "settings/dev",
        "stackprefix": "app-dev",
        "s3bucket": "app-dev-bucket"
    ],

    "test": [
        "AccountID": "11112",
        "AssumeRole": "app-test-role",
        "folderpath": "settings/test",
        "stackprefix": "app-test",
        "s3bucket": "app-test-bucket"
    ],

    "stage": [
        "AccountID": "11113",
        "AssumeRole": "app-stage-role",
        "folderpath": "settings/stage",
        "stackprefix": "app-stage",
        "s3bucket": "app-stage-bucket"
    ],

    "prod": [
         "AccountID": "11114",
        "AssumeRole": "app-prod-role",
        "folderpath": "settings/prod",
        "stackprefix": "app-prod",
        "s3bucket": "app-prod-bucket"
    ]
]
    
    pipeline {
    agent any
    tools {
        jdk 'JDK-21'
        maven 'Maven-3.9.9'
    }
    options {
        // Don't allow two builds at the same time
        disableConcurrentBuilds()

        // Keep only the last 5 builds
        buildDiscarder(
            logRotator(
                numToKeepStr: '5'
            )
        )

        // Add time to console logs
        timestamps()

        // Stop the pipeline after 25 minutes
        timeout(time: 25, unit: 'MINUTES')
    }
    
    parameters {

        choice(
            name: 'SERVER',
            choices: ['server-1', 'server-2', 'server-3'],
            description: 'Select a server'
        )
        choice(
        name: 'REGION',
        choices: ['eu-west-1', 'eu-west-2', 'us-east-1'],
        description: 'Select AWS region'
    )
        booleanParam(
        name: 'TAKE_BACKUP',
        defaultValue: true,
        description: 'Do you want to take a backup?'
    )


    }
    environment {
    APP_NAME = 'server-app'
    SERVER_NAME = "${params.SERVER}"
    AWS_REGION = "${params.REGION}"
    BACKUP = "${params.TAKE_BACKUP}"
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

        dir('self-testing-project') {
            git branch: 'self-testing2',
                url: 'https://github.com/ambreen87/jenkins-lab1.git'
        }
    }
}
          

        stage('Show Selection') {

        steps {

            echo "Selected Region: ${params.REGION}"
        
        }
    
    }
        stage('Show Backup') {
            steps {
        echo "Take Backup: ${params.TAKE_BACKUP}"
    }
}
        stage("call restart application"){
            steps {
                script{
                echo "calling restart application pipeline"
                def restartBuild = build(
                job: 'restart-application',
                wait: true,
                propagate: false
            )

            echo "Restart Pipeline Result: ${restartBuild.result}"

            if (restartBuild.result == 'SUCCESS') {
                echo "Restart application completed successfully"
            } else {
                echo "Restart application failed"
            }
        }
    }
}
                
                
    }
}    
