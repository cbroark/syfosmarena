#!/usr/bin/env groovy

pipeline {
    agent any

     tools {
                jdk 'openjdk11'
           }

     environment {
           APPLICATION_NAME = 'syfosmarena'
           DOCKER_SLUG = 'syfo'
       }

     stages {
         stage('initialize') {
                    steps {
                        init action: 'default'
                        script {
                            sh(script: './gradlew clean')
                            def applicationVersionGradle = sh(script: './gradlew -q printVersion', returnStdout: true).trim()
                            env.APPLICATION_VERSION = "${applicationVersionGradle}-${env.COMMIT_HASH_SHORT}"
                            if (applicationVersionGradle.endsWith('-SNAPSHOT')) {
                                env.APPLICATION_VERSION = "${applicationVersionGradle}.${env.BUILD_ID}-${env.COMMIT_HASH_SHORT}"
                            } else {
                                env.DEPLOY_TO = 'production'
                            }
                            init action: 'updateStatus', applicationName: env.APPLICATION_NAME, applicationVersion: env.APPLICATION_VERSION
                        }
                    }
                }
        stage('build') {
            steps {
                sh './gradlew build -x test'
            }
        }
        stage('run tests (unit & intergration)') {
            steps {
                sh './gradlew test'
                slackStatus status: 'passed'
            }
        }
        stage('create uber jar') {
            steps {
                sh './gradlew shadowJar'
            }
        }
         stage('deploy to preprod') {
             steps {
                     dockerUtils action: 'createPushImage'
                     deployApp action: 'kubectlDeploy', cluster: 'preprod-fss'
                 }
             }
         stage('deploy to production') {
             when { environment name: 'DEPLOY_TO', value: 'production' }

             steps {
                     deployApp action: 'kubectlDeploy', cluster: 'prod-fss', file: 'naiserator-prod.yaml'
                 }
             }
        }
        post {
            always {
                postProcess action: 'always'
            }
            success {
                postProcess action: 'success'
            }
            failure {
                postProcess action: 'failure'
            }
        }
}
