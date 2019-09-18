#!groovy

/**
 * This Jenkinsfile is intended to run on http://jencov.us.rdlabs.hpecorp.net and may fail
 * anywhere else. It makes assumptions about plugins being installed, labels mapping to nodes
 * that can build what is needed, etc.
 *
 * Copyright 2019 Hewlett Packard Enterprise Development LP., All Rights Reserved
 *
 * @author : vinay.makam-anjaneya@hpe.com
 *    Preparation : prepare build environment (Git, ...)
 *    Build       : execute build tasks for Fast-data project
 *    UnitTests   : execute configured Unit Tests
 *    Analyze     : static analysis of source code using Coverity
 *    Package     : generate packages (rpm, docker, ...) and sign them
 */

/* Declarative pipeline must be enclosed within a pipeline block */
pipeline {

    // agent section specifies where the entire Pipeline will execute in the Jenkins environment
    agent any

    /**
     * node allows for additional options to be specified you can also specify label '' without
     * the node option if you want to execute the pipeline on any available agent use the option
     * 'agent any'
     */
    options {
        buildDiscarder(
                // Only keep the 05 most recent builds
                logRotator(numToKeepStr: '05'))
        disableConcurrentBuilds()
        skipStagesAfterUnstable()
        parallelsAlwaysFailFast()
        timeout(time: 90, unit: 'MINUTES')
    }

    environment {
        PROJECT_NAME = 'Hackathon_Calculator'
        COVERITY_HOST = 'jencov.us.rdlabs.hpecorp.net'
        COVERITY_PORT = '9090'
        COVERITY = credentials('coverity_credentials')
        VIRTUAL_ENV = "${env.WORKSPACE}/venv"
    }

    /*
     * stages contain one or more stage directives
     */
    stages {
        /**
         * The stage directive should contain a steps section, an optional agent section, or
         * other stage-specific directives all of the real work done by a Pipeline will be
         * wrapped in one or more stage directives
         */
        stage('Preparation') {
            /**
             * steps section defines a series of one or more steps to be executed in a given
             * stage directive
             */
            steps {
                sh """
                    echo ${SHELL}
                    [ -d venv ] && rm -rf venv
                    python3.6 -m venv venv
                    export PATH=${VIRTUAL_ENV}/bin:${PATH}
		    pip install --upgrade pip
                    pip install -r requirements.txt
                """
            }
        }

        stage('flake8') {
            steps {
                sh """
		#. venv/bin/activate
		export PATH=${VIRTUAL_ENV}/bin:${PATH}
		flake8 --exclude=venv* --statistics  --exit-zero . | tee flake8.log || true
		"""
            }
        }
        
        stage('Pylint') {
            steps {
                sh """
		export PATH=${VIRTUAL_ENV}/bin:${PATH}
		pylint ${WORKSPACE}/src --disable=C irisvmpy | tee pylint.log || true
		"""
            }
        }

        stage('Unit tests') {
            steps {
                sh """
		export PATH=${VIRTUAL_ENV}/bin:${PATH}
		pytest ${WORKSPACE}/src/ 
		py.test -svv --cov=src/ --cov-report=term-missing | tee pytest.log || true
                """
            }
        }
    }
    /**
     * post section defines actions which will be run at the end of the Pipeline run or stage
     * post section condition blocks: always, changed, failure, success, unstable, and aborted
     */
    post {
        always {
            script {
                    NotifyBuild(currentBuild.result)
                }
            }
        }
    }

def NotifyBuild(String BuildStatus) {
    // build status of null means successful
    BuildStatus = BuildStatus ?: 'SUCCESSFUL'

    // Default values
    def subject = "[${BuildStatus}] - $PROJECT_NAME - Build # $BUILD_NUMBER"

    def committerEmail = sh(
            script: "cd \"${WORKSPACE}/src/\" && git --no-pager show -s --format=\'%ae\'",
            returnStdout: true
    ).trim()

    // Send notifications
    emailext(
            subject: subject,
            body: '''${SCRIPT, template="allure-report.groovy"}''',
            recipientProviders: [[$class: 'CulpritsRecipientProvider'],
                                 [$class: 'RequesterRecipientProvider']],
            to: committerEmail
    )
}
