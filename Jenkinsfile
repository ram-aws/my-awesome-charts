#!/usr/bin/env groovy
import org.apache.commons.lang.StringUtils
import java.util.regex.Matcher
import java.util.regex.Pattern

// aws-nextBot-pipeline

pipeline {
    agent { docker { image 'python:3.10.1-alpine' } }

    options {
        timestamps()
        timeout(time: 48, unit: 'HOURS')
        parallelsAlwaysFailFast()
    }

    parameters {
        string(name: 'APP_NAME', defaultValue: "custodian", description: "Application name")
        string(name: 'ENV', defaultValue: "dev")
        booleanParam(name: 'IS_PREMERGE', defaultValue: "false")

        booleanParam(name: 'IS_C7N_PLUGIN_CHANGE', defaultValue: false, description: 'Is a c7n plugin build')
        booleanParam(name: 'IS_C7N_POLICY_CHANGE', defaultValue: false, description: 'Is a c7n policy build')
        booleanParam(name: 'IS_C7N_SCP_CHANGE', defaultValue: false, description: 'Is a c7n scp build')
        booleanParam(name: 'IS_SLS_CHANGE', defaultValue: false, description: 'Is a serverless build')
        booleanParam(name: 'IS_AMPLIFY_CHANGE', defaultValue: false, description: 'Is an amplify build')
        booleanParam(name: 'IS_QUALMAP_CHANGE', defaultValue: false, description: 'Is a qualmap build')
        booleanParam(name: 'IS_SINGLE_ACCOUNT_CHANGE', defaultValue: false, description: 'Is a single account change')
        booleanParam(name: 'IS_ACCOUNTS_IAM_CHANGE', defaultValue: false, description: 'Is an account iam change')
        booleanParam(name: 'IS_DASHBOARD_CHANGE', defaultValue: false, description: 'Is a compliance dashboard change')

        string(name: 'PLUGINS_CHANGED', defaultValue: "", description: 'Plugin that has changed')
        string(name: 'POLICIES_CHANGED', defaultValue: "", description: 'Policy resources that have changed')
        string(name: "SLS_APPS_CHANGED", defaultValue: "", description: 'SLS apps that have changed')
        booleanParam(name: "SLS_MULTIACCOUNT", defaultValue: false, description: 'deploy SLS to all accounts')

        string(name: "BRANCH", defaultValue: "feature/ABGN-3695", description: "Git branch to be deployed")
        string(name: "BEHAVE_TAGS", defaultValue: "integration_test", description: "Run particular features having the test tag")

        string(name: 'CLOUDX_GIT_REPO', defaultValue: "https://sourcecode.jnj.com/scm/asx-najf/awsapi.git", description: "Git Repo for Cloudx Apis")
        string(name: 'KEYS_GIT_REPO', defaultValue: "https://sourcecode.jnj.com/scm/asx-nasl/python.git", description: "Git Repo for Cloudx private keys")
        string(name: 'GIT_CREDENTIAL_ID', defaultValue: "c7faad16-7446-451b-ad77-556dcd974748", description: "The credential id of the Jenkins Credential of the user that is used to download private keys.")

        booleanParam(name: 'RUN_BEHAVE_TEST', defaultValue: true, description: 'Run Behave Test')
        booleanParam(name: 'RUN_ALL_C7N_BEHAVE_TESTS', defaultValue: false, description: 'Run All Behave Tests')
        booleanParam(name: 'RUN_UNIT_TEST', defaultValue: true, description: 'Run Unit Test')
        booleanParam(name: 'RUN_LINT', defaultValue: true, description: 'Run Lint')

        string(name: 'Region', defaultValue: "us-east-1", description: 'AWS region')
        string(name: 'ProjectId', defaultValue: "", description: 'ProjectId that has been added to the org')
        string(name: 'PYLINT_REGEX', defaultValue: "(Your code has been rated at )[\\s]*([0-7].\\d*)(\\/10)", description: "Regex expression used to validated lint score")
        string(name: 'UNIT_TEST_REGEX', defaultValue: "FAILED ([(]failures=[0-9]*[)]|[(]errors=[0-9]*[)]|[(]errors=[0-9]*, failures=[0-9]*[)])", description: "Regex expression used to validated unit test execution")
        string(name: 'BDD_FEATURE_REGEX', defaultValue: '([1-9][0-9]*)\\s(feature)(s)*[\\s](passed),\\s(0)\\s(failed,)\\s([0-9]*)\\s(skipped)', description: "Regex expression used to validate Behave execution summary for features")
        string(name: 'BDD_SCENARIO_REGEX', defaultValue: "([1-9][0-9]*)\\s(scenario)(s)*[\\s](passed),\\s(0)\\s(failed,)\\s([0-9]*)\\s(skipped)", description: "Regex expression used to validate Behave execution summary for scenarios")
        string(name: 'BDD_STEP_REGEX', defaultValue: "([1-9][0-9]*)\\s(step)(s)*[\\s](passed),\\s(0)\\s(failed,)\\s([0-9]*)\\s(skipped)", description: "Regex expression used to validate Behave execution summary for steps")
        string(name: 'BEHAVE_REGEX_NEGETIVE', defaultValue: "(Exception AmbiguousStep:|Failing scenarios:)", description: "Regex expression used to validate Behave execution failure")
        string(name: 'COVERAGE_REGEX', defaultValue: '(TOTAL)[\\s\\d]*([8-9]\\d%|\\d{3,}%)$', description: "Regex expression used to validate Code Coverage")
        string(name: 'CUSTODIAN_VALIDATE_REGEX', defaultValue: '(custodian.commands:ERROR)', description: "Regex expression used to catch Custodian policy validation")
        string(name: 'C7N_ORG_REGEX', defaultValue: '(c7n_org:WARNING Error)', description: "Regex expression used to catch C7N-Org failures")
        string(name: 'ACCESS_DENIED_REGEX', defaultValue: '(AccessDenied|Access Denied)', description: "Regex expression used to catch iam permissions errors")

        // Defines whether to use separate EC2 to deploy C7N
        booleanParam(name: "C7N_EC2_RAPID_DEPLOY", defaultValue: false, description: 'Deploy C7N using separate EC2 to improve performance')
        string(name: 'C7N_EC2_DEPLOY_SERVER', defaultValue: "AWSALXNVA1059", description: "Hostname for C7N code deploy")
        string(name: 'EC2_INSTANCE_ID', defaultValue: "i-073c774e129abc592", description: "Instance ID for C7N code deploy")
        string(name: 'EC2_INSTANCE_ACCOUNT', defaultValue: "itx-alx", description: "Account ID for C7N EC2 instance")
        string(name: 'SSH_CREDENTIAL_ID', defaultValue: "6ecf9d79-b086-427f-89a8-fa06a5dc1754", description: "Jenkins Credential ID used to log in to deployment server.")
        string(name: 'USER_NAME', defaultValue: "sa-its-cdx-api-user", description: "User to log into EC2 build server.")
        string(name: 'APP_DEPLOY_DIR', defaultValue: "/app/xbot/deploy", description: "App directory to use for deploys")

        // Defines batch size for account deploy
        string(name: 'DEPLOY_BATCH_SIZE', defaultValue: "10", description: "Defines number of accounts in an accounts*.yml file.")

    }
    environment {
        LC_ALL = "en_US.utf-8"
        LANG = "en_US.utf-8"
        PATH = "${HOME}/.local/bin:${WORKSPACE}/serverless/node_modules/.bin:${WORKSPACE}/qualmap/webapp/node_modules/.bin:${env.PATH}"
        sddcapi_boot_dir = "${WORKSPACE}/ext_config"
        PLUGINS_CHANGED = "${params.PLUGINS_CHANGED}"
        POLICIES_CHANGED = "${params.POLICIES_CHANGED}"
        SLS_APPS_CHANGED = "${params.SLS_APPS_CHANGED}"
        IS_QUALMAP_CHANGE = "${params.IS_QUALMAP_CHANGE}"
        IS_SINGLE_ACCOUNT_CHANGE = "${params.IS_SINGLE_ACCOUNT_CHANGE}"
        IS_ACCOUNTS_IAM_CHANGE = "${params.IS_ACCOUNTS_IAM_CHANGE}"
        ENV = "${params.ENV}"
        NVM_DIR="${HOME}/.nvm"
        RUN_UNIT_TEST="${params.RUN_UNIT_TEST}"
        RUN_LINT="${params.RUN_LINT}"
        SLS_MULTIACCOUNT = false
        EC2_INSTANCE_ID = "${params.EC2_INSTANCE_ID}"
        EC2_INSTANCE_ACCOUNT = "${params.EC2_INSTANCE_ACCOUNT}"
    }

    stages {
        stage("SCM checkout") {
            steps {
                script {
                    ansiColor('xterm') {
                        currentBuild.displayName = "#${BUILD_ID}-${params.BRANCH}-${params.APP_NAME}-${params.ProjectId}"
                        currentBuild.description = "${params.APP_NAME} deployed on the container"
                        checkout changelog: false, poll: false, scm: [$class: 'GitSCM', branches: [[name: params.BRANCH]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: '.'], [$class: 'CleanBeforeCheckout']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: params.GIT_CREDENTIAL_ID, url: params.CLOUDX_GIT_REPO]]]
                        checkout changelog: false, poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'keys'], [$class: 'CleanBeforeCheckout']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: params.GIT_CREDENTIAL_ID, url: params.KEYS_GIT_REPO]]]
                        step([$class: 'StashNotifier'])
                        success "Copying done"
                    }
                }
            }
        }

        stage('Credential and Env Setup') {
            steps {
                script {
                    ansiColor('xterm') {
                        info "Configuring private keys in build environment"
                        sh '''#!/bin/bash
                        set -e

                        . jenkins/c7n_scripts/deploy-keys.sh ${WORKSPACE} $ENV
                        ls -l ${WORKSPACE}/ext_config
                        '''
                        info "Installing common python dependencies"
                        sh '''#!/bin/bash
                        set -e

                        . jenkins/setup_python.sh ${WORKSPACE}/jenkins $ENV
                        '''
                        info "Copying AWS credential config to Jenkins user"
                        sh '''#!/bin/bash
                        set -e

                        mkdir -p /home/jenkins/.aws
                        cp ${WORKSPACE}/jenkins/c7n_scripts/aws-config /home/jenkins/.aws/config
                        '''
                        info "Updating boot_config.ini"
                        sh '''#!/bin/bash
                        set -e

                        cd ${WORKSPACE}/jenkins/c7n_scripts/
                        sh update-boot-config.sh ${WORKSPACE} custodian
                        '''
                        info "Installing C7N deploy script dependencies"
                        installCustodianDeployScriptPipEnv()
                        info "Retrieving temporary credentials"			
                        sh '''#!/bin/bash
                        set -e

                        cd ${WORKSPACE}/jenkins/c7n_scripts/
                        pipenv run python get_temp_aws_creds.py -e $ENV
                        '''
                    }
                }
            }
        }

        stage('Node Setup') {
            when {
                expression { return params.IS_SLS_CHANGE || params.IS_AMPLIFY_CHANGE || params.IS_QUALMAP_CHANGE}
            }
            steps {
                ansiColor('xterm') {
                    sh '''#!/bin/sh
                    shopt -s nullglob
                    set -e

                    wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
                    source /home/jenkins/.bashrc
                    nvm install --lts --no-progress --latest-npm
                    . ~/.nvm/nvm.sh

                    nvm use --lts
                    which node
                    which npm

                    npm install -g serverless @aws-amplify/cli
                    '''
                }
            }
        }

        stage('C7N') {
            when {
                expression { return params.IS_C7N_PLUGIN_CHANGE || params.IS_C7N_POLICY_CHANGE || params.IS_SINGLE_ACCOUNT_CHANGE}
            }
            stages{
                stage("C7N EC2 Startup") {
                    when {
                        expression { return params.C7N_EC2_RAPID_DEPLOY && params.ENV.toUpperCase() == "PROD" }
                    }
                    steps {
                        script {
                            ansiColor('xterm') {
                            sh '''#!/bin/bash
                            set -e

                            cd ${WORKSPACE}/jenkins/c7n_scripts/
                            pipenv run python get_temp_aws_creds.py -a $EC2_INSTANCE_ACCOUNT -e $ENV
                            aws ec2 start-instances --instance-ids ${EC2_INSTANCE_ID}
                            '''   
                            }
                        }
                    }
                }
                stage("C7N Python Install") {
                    steps {
                        script {
                            ansiColor('xterm') {
                                installCustodianPipEnv()
                            }
                        }
                    }
                }
                stage('C7N Get Env Accounts and Creds') {
                    steps{
                        script{
                            ansiColor('xterm') {
                                def project_id = params.ProjectId ? params.ProjectId : "N/A"
                                def new_regions = params.Region ? params.Region : "N/A"
                                info "Creating accounts.yml files"
                                sh '''#!/bin/sh
                                set -e

                                cd ${WORKSPACE}/jenkins/c7n_scripts/
                                pipenv run python create-accounts.py $ENV . \
                                --acct ''' + project_id + ''' \
                                --region ''' + new_regions + ''' \
                                --batch ''' + params.DEPLOY_BATCH_SIZE + ''' \
                                '''
                                sh '''#!/bin/sh
                                set -e

                                cd ${WORKSPACE}/jenkins/c7n_scripts/
                                cat *_accounts.yml
                                ''' 
                                info "Creating accounts_global.yml files"
                                sh '''#!/bin/sh
                                set -e

                                cd ${WORKSPACE}/jenkins/c7n_scripts/
                                pipenv run python create-accounts.py $ENV . \
                                --globalservice \
                                --acct ''' + project_id + ''' \
                                --region ''' + new_regions + ''' \
                                --batch ''' + params.DEPLOY_BATCH_SIZE + ''' \
                                '''
                                sh '''#!/bin/sh
                                set -e

                                cd ${WORKSPACE}/jenkins/c7n_scripts/
                                cat *_accounts_global.yml
                                ''' 
                                info "Renewing credentials"
                                sh '''#!/bin/sh
                                set -e

                                cd ${WORKSPACE}/jenkins/c7n_scripts/
                                pipenv run python get_temp_aws_creds.py -e $ENV
                                '''
                            }
                        }
                    }
                }
                stage('C7N PyLint'){
                    when {
                        expression { return (params.IS_C7N_POLICY_CHANGE || params.IS_C7N_PLUGIN_CHANGE) && params.RUN_LINT }
                    }
                    steps{
                        script{
                            echo "C7N: Running PyLint"
                            ansiColor('xterm'){
                                sh '''#!/bin/sh
                                    shopt -s nullglob
                                    set -e

                                    cd ${WORKSPACE}/custodian/
                                    pylint --rcfile=${WORKSPACE}/pylintrc.cfg ${WORKSPACE}/custodian > pylint_plugins.log || true
                                    cat pylint_plugins.log
                                    echo "Finished Pylint execution"
                                    '''
                                findPattern(params.PYLINT_REGEX, true, params.RUN_LINT)
                            }
                        }
                    }
                }
                stage('C7N Unit Tests') {
                    when {
                        expression { return params.IS_C7N_PLUGIN_CHANGE && params.RUN_UNIT_TEST }
                    }
                    steps {
                        script {
                            ansiColor('xterm') {
                                info "Beginning unit test execution for C7N plugins."
                                info "Plugins changed ${params.PLUGINS_CHANGED}"
                                def pluginArray = params.PLUGINS_CHANGED.split()
                                def pluginSet = pluginArray as Set
                                if ('utils' in pluginSet) {
                                    info "Detected utils change.  Need to run all unit tests."
                                    sh '''#!/bin/sh
                                    set -e

                                    cd ${WORKSPACE}/custodian/plugins/jnj_c7n_plugins/
                                    for plugin in *
                                    do
                                        if [[ -d $plugin ]]
                                        then
                                            cd $plugin
                                            if [[ -d ./tests ]]
                                            then
                                                cd ${WORKSPACE}/custodian
                                                pipenv run python -m pytest ${WORKSPACE}/custodian/plugins/jnj_c7n_plugins/$plugin/tests/
                                                cd ${WORKSPACE}/custodian/plugins/jnj_c7n_plugins/
                                            fi
                                        fi
                                    done
                                    '''
                                } else {
                                    info "No utils change detected.  Running case by case unit tests."
                                    for (plugin in pluginSet) {
                                        info "Running unit tests for ${plugin}."
                                        sh '''#!/bin/sh
                                        set -e

                                        cd ${WORKSPACE}/custodian
                                        pipenv run python -m pytest ${WORKSPACE}/custodian/plugins/jnj_c7n_plugins/''' + plugin + '''/tests
                                        '''
                                    }
                                }
                                findPattern(params.UNIT_TEST_REGEX, true, params.RUN_UNIT_TEST)
                                success "Completed C7N unit testing."
                            }
                        }
                    }
                }
                stage('Update Metadata') {
                    when {
                        expression { return params.IS_C7N_POLICY_CHANGE }
                    }
                    steps {
                        script {
                            info "Adding service in DynamoDB for whitelisting"
                            ansiColor('xterm') {
                                def policyArray = params.POLICIES_CHANGED.split()
                                def policySet = policyArray as Set
                                policySet = policySet - ["bdd_utils", "utils"]
                                for (resource in policySet) {
                                    sh '''#!/bin/sh
                                        shopt -s nullglob
                                        set -e

                                        RESOURCE='''+resource+'''

                                        cd ${WORKSPACE}/custodian/policies/$RESOURCE
                                        echo "Updating ddb to add/update $RESOURCE service"
                                        python3.6 ${WORKSPACE}/jenkins/c7n_scripts/update-appsync.py -e $ENV -c ${WORKSPACE}/jenkins/deployment_targets.yaml -d .
                                        '''
                                }
                            }
                        }
                    }
                }
                stage('C7N BDD') {
                    when {
                        expression { return (params.IS_C7N_POLICY_CHANGE || params.IS_C7N_PLUGIN_CHANGE) && params.RUN_BEHAVE_TEST }
                    }
                    steps {
                        script {
                            ansiColor('xterm') {
                                info "Beginning bdd execution for C7N plugins."
                                def pluginArray = params.PLUGINS_CHANGED.split()
                                def pluginSet = pluginArray as Set
                                def policyArray = params.POLICIES_CHANGED.split()
                                def policySet = policyArray as Set
                                policySet = policySet - ["bdd_utils", "utils"]
                                def resourceSet = pluginSet + policySet
                                info "Plugins changed: ${pluginSet}. Policies changed: ${policySet}. Resources changed: ${resourceSet}"
                                if('utils' in pluginSet) {
                                    info "Detected utils change.  Need to run all bdd tests."
                                    sh '''#!/bin/sh
                                    shopt -s nullglob
                                    set -e

                                    cd ${WORKSPACE}/custodian/policies/
                                    for resource in *
                                    do
                                        if [[ -d $resource && $resource != "bdd_utils" && $resource != "utils" ]]
                                        then
                                            cd $resource
                                            if [[ -d ./bdd ]]
                                            then
                                                cd ${WORKSPACE}/custodian
                                                pipenv run python -m behave policies/${resource}/bdd
                                                cd ${WORKSPACE}/custodian/policies/
                                            else
                                                echo "custodian/policies/$resource/bdd not present. Exiting with failure"
                                                exit 1
                                            fi
                                        fi
                                    done
                                    '''
                                } else {
                                    info "No utils change detected.  Running case by case bdd."
                                    for (resource in resourceSet) {
                                        info "Running unit tests for ${resource}."
                                        sh '''#!/bin/sh
                                        shopt -s nullglob
                                        set -e

                                        RESOURCE='''+resource+'''

                                        cd ${WORKSPACE}/custodian/policies/$RESOURCE
                                        pwd
                                        if [[ -d ./bdd ]]
                                        then
                                            cd ${WORKSPACE}/custodian
                                            pipenv run python -m behave policies/${RESOURCE}/bdd --tags='''+params.BEHAVE_TAGS+'''
                                            cd ${WORKSPACE}/custodian/policies/
                                        else
                                            echo "custodian/policies/$RESOURCE/bdd not present. Exiting with failure"
                                            exit 1
                                        fi
                                        '''
                                    }
                                }
                                info "Listing directory for ${WORKSPACE}/custodian/policies"
                                def dlist = listDirs("${WORKSPACE}/custodian/policies")
                                if (dlist.size() == 1) {
                                    info "Only bdd_utils present, no need to check for regex match."
                                } else {
                                    findPattern(params.BDD_FEATURE_REGEX, false, true)
                                    findPattern(params.BDD_SCENARIO_REGEX, false, true)
                                    findPattern(params.BDD_STEP_REGEX, false, true)
                                }
                                success "Completed C7N BDD."
                            }
                        }
                    }
                }
                stage('C7N Deploy') {
                    when {
                        expression { return params.IS_C7N_PLUGIN_CHANGE || params.IS_C7N_POLICY_CHANGE || params.IS_SINGLE_ACCOUNT_CHANGE}
                    }
                    steps{
                        script{
                            echo "C7N: Deploying C7N Plugin"
                            ansiColor('xterm') {
                                String[] dirs = WORKSPACE.split("/"); // Used to isolate the build directory
                                workDir = dirs[dirs.length-1]; // Current build directory
                                info "workDir ${workDir}"
                                if (params.C7N_EC2_RAPID_DEPLOY) {
                                    sshagent(credentials: ["${params.SSH_CREDENTIAL_ID}"]) {
                                            sh "ssh -o StrictHostKeyChecking=no ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER} \"sudo rm -rf ${params.APP_DEPLOY_DIR}\""
                                            sh "ssh -o StrictHostKeyChecking=no ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER} \"sudo rm -rf ${WORKSPACE}/shared_config\""
                                            sh "ssh -o StrictHostKeyChecking=no ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER} \"sudo rm -rf ${WORKSPACE}/custodian/config\""
                                            sh "ssh -o StrictHostKeyChecking=no ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER} \"sudo rm -rf /root/.local/share/virtualenvs\""
                                            sh "ssh -o StrictHostKeyChecking=no ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER} \"sudo mkdir -p -m 777 ${params.APP_DEPLOY_DIR} && sudo mkdir -p -m 777 ${WORKSPACE}/shared_config\""
                                            sh "ssh -o StrictHostKeyChecking=no ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER} \"sudo mkdir -p -m 777 ${params.APP_DEPLOY_DIR} && sudo mkdir -p -m 777 ${WORKSPACE}/custodian/config\""
                                            sh "scp -r -o StrictHostKeyChecking=no ${WORKSPACE} ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER}:${params.APP_DEPLOY_DIR}"
                                            sh "scp -r -o StrictHostKeyChecking=no ${WORKSPACE}/shared_config/* ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER}:${WORKSPACE}/shared_config"
                                            sh "scp -r -o StrictHostKeyChecking=no ${WORKSPACE}/custodian/config/* ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER}:${WORKSPACE}/custodian/config"
                                            sh "ssh -o StrictHostKeyChecking=no ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER} \"sudo ls -al ${WORKSPACE}/shared_config && sudo ls -al ${params.APP_DEPLOY_DIR}\""
                                            sh "ssh -o StrictHostKeyChecking=no ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER} \"sudo chmod -R 777 ${params.APP_DEPLOY_DIR}\""
                                            sh "ssh -o StrictHostKeyChecking=no ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER} \"${params.APP_DEPLOY_DIR}/${workDir}/jenkins/setup_python.sh ${params.APP_DEPLOY_DIR}/${workDir}/jenkins ${params.ENV}\""
                                            sh "ssh -o StrictHostKeyChecking=no ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER} \"rm -rf ${params.APP_DEPLOY_DIR}/${workDir}/ext_config/boot_config.ini\""
                                            sh "ssh -o StrictHostKeyChecking=no ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER} \"${params.APP_DEPLOY_DIR}/${workDir}/jenkins/c7n_scripts/deploy-keys.sh ${params.APP_DEPLOY_DIR}/${workDir} ${params.ENV}\""
                                            sh "ssh -o StrictHostKeyChecking=no ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER} \"${params.APP_DEPLOY_DIR}/${workDir}/jenkins/c7n_scripts/update-boot-config.sh ${params.APP_DEPLOY_DIR}/${workDir} custodian\""
                                    }
                                    installCustodianPipEnvOnEc2("${params.APP_DEPLOY_DIR}/${workDir}/custodian", "${params.ENV}")
                                    installCustodianDeployScriptPipEnvOnEc2("${params.APP_DEPLOY_DIR}/${workDir}/jenkins/c7n_scripts")
                                }

                                def pluginArray = params.PLUGINS_CHANGED.split()
                                def pluginSet = pluginArray as Set
                                def policyArray = params.POLICIES_CHANGED.split()
                                def policySet = policyArray as Set

                                def resourceSet = pluginSet + policySet - "bdd_utils"

                                valuesYaml = readYaml (file: './jenkins/deployment_targets.yaml')
                                config = valuesYaml."aws"."${params.ENV}"
                                custodian_bucket = config["custodian_bucket"]

                                if('utils' in pluginSet || params.IS_SINGLE_ACCOUNT_CHANGE ) {
                                    info ("Plugins change or single account change.  Deploying all policies.")

                                    sh '''
                                        cd ${WORKSPACE}/custodian/policies
                                        ls -d */ >> resources.txt
                                    '''
                                    def resources_file = readFile "./custodian/policies/resources.txt"
                                    def services = resources_file.split("\n")

                                    for(service in services) {
                                        if(service != "bdd_utils/" && service != "utils/" && service != "__pycache__/") {
                                            metadataYAML = readYaml(file: './custodian/policies/'+service+'/service_metadata.yml')
                                            try {
                                                global = metadataYAML."Service"."GlobalService"
                                            } catch(exc) {
                                                global = "False"
                                            }
                                            if(global == "False" || global == null) {
                                                sh '''#!/bin/sh
                                                echo accounts ''' + service + ''' >> ${WORKSPACE}/jenkins/c7n_scripts/deploy_list.txt
                                                '''
                                            }
                                            else{
                                                sh '''#!/bin/sh
                                                echo accounts_global ''' + service + ''' >> ${WORKSPACE}/jenkins/c7n_scripts/deploy_list.txt
                                                '''
                                            }
                                        }
                                    }
                                }else{
                                    for (resource in resourceSet) {
                                        metadataYAML = readYaml(file: './custodian/policies/'+resource+'/service_metadata.yml')
                                        try {
                                            global = metadataYAML."Service"."GlobalService"
                                        } catch(exc) {
                                            global = "False"
                                        }                                        
                                        if(global == "False" || global == null) {
                                            sh '''#!/bin/sh
                                            echo accounts ''' + resource + ''' >> ${WORKSPACE}/jenkins/c7n_scripts/deploy_list.txt
                                            '''
                                        }
                                        else{
                                            sh '''#!/bin/sh
                                            echo accounts_global ''' + resource + ''' >> ${WORKSPACE}/jenkins/c7n_scripts/deploy_list.txt
                                            '''
                                        }
                                    }
                                }
                                // Deploy everything added to the deploy list
                                info "Deploy list"
                                sh '''#!/bin/sh

                                cat ${WORKSPACE}/jenkins/c7n_scripts/deploy_list.txt
                                '''
                                if (params.C7N_EC2_RAPID_DEPLOY) {
                                   sshagent(credentials: ["${params.SSH_CREDENTIAL_ID}"]) {
                                       info "Copying deploy list to EC2"
                                        sh "scp -v -o StrictHostKeyChecking=no ${WORKSPACE}/jenkins/c7n_scripts/deploy_list.txt ${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER}:${params.APP_DEPLOY_DIR}/${workDir}/jenkins/c7n_scripts/deploy_list.txt"
                                   }
                                   deployC7nOnEC2(custodian_bucket, workDir, "${params.APP_DEPLOY_DIR}/${workDir}/jenkins/c7n_scripts/deploy_list.txt")
                                } else {
                                   deployC7nOnAgent(custodian_bucket, "${WORKSPACE}/jenkins/c7n_scripts/deploy_list.txt")
                                }
                            }
                        }
                    }
                }
                stage('C7N Post Deploy') {
                    when {
                        expression { return params.IS_C7N_PLUGIN_CHANGE || params.IS_C7N_POLICY_CHANGE || params.IS_SINGLE_ACCOUNT_CHANGE}
                    }
                    steps{
                        sh '''#!/bin/sh
                        shopt -s nullglob
                        set -e

                        cd ${WORKSPACE}/jenkins/c7n_scripts
                        if [[ -f disable-cwe.py ]]
                        then
                            pipenv run python disable-cwe.py -e $ENV
                        fi
                        '''
                    }
                }
                stage("C7N EC2 Shutdown") {
                    when {
                        expression { return params.C7N_EC2_RAPID_DEPLOY && params.ENV.toUpperCase() == "PROD" }
                    }
                    steps {
                        script {
                            ansiColor('xterm') {
                            sh '''#!/bin/bash
                            set -e

                            cd ${WORKSPACE}/jenkins/c7n_scripts/
                            pipenv run python get_temp_aws_creds.py -a $EC2_INSTANCE_ACCOUNT -e $ENV
                            aws ec2 stop-instances --instance-ids ${EC2_INSTANCE_ID}
                            '''   
                            }
                        }
                    }
                }
            }
        }

        stage('SLS') {
            when {
                expression { return params.IS_SLS_CHANGE && !params.IS_DASHBOARD_CHANGE }
            }
            stages{
                stage('SLS Run PyLint') {
                    when {
                        expression { return params.RUN_LINT }
                    }
                    steps{
                        echo "SLS: Running PyLint"
                        ansiColor('xterm') {
                            sh '''#!/bin/sh
                            shopt -s nullglob
                            set -e

                            cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED
                            find . -type f -name "*.py" | xargs pylint --rcfile=${WORKSPACE}/pylintrc.cfg . > pylint.log || true
                            cat pylint.log
                            echo "Finished Pylint execution"
                            '''
                            findPattern(params.PYLINT_REGEX, true, params.RUN_LINT)
                        }
                    }
                }
                stage('SLS Run Unit Test') {
                    when {
                        expression { return params.RUN_UNIT_TEST }
                    }
                    steps{
                        echo "SLS: Running Unit Test"
                        script{
                            ansiColor('xterm') {
                                def slsWorkDir = "${WORKSPACE}/serverless/${params.SLS_APPS_CHANGED}/"
                                pipInstall(slsWorkDir,"requirements.txt")

                                sh '''#!/bin/sh
                                shopt -s nullglob
                                set -e

                                cd ${WORKSPACE}/jenkins/c7n_scripts
                                sh ./update-boot-config-sls.sh ${WORKSPACE} $SLS_APPS_CHANGED

                                cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/lambda
                                pwd
                                ls -la
                                for i in *; do
                                    if [[ -d $i ]]
                                    then
                                        echo "$i is a directory"
                                        cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/lambda/$i/tests
                                        pytest
                                        cd -  
                                        # python3.6 -m unittest
                                    fi
                                done
                                '''
                                findPattern(params.UNIT_TEST_REGEX, true, params.RUN_UNIT_TEST)
                            }
                        }
                    }
                }
                stage('SLS get credentials') {
                    steps{
                        echo "SLS: getting credentials"
                        script{
                            ansiColor('xterm') {
                                if(!params.IS_SINGLE_ACCOUNT_CHANGE) {
                                    sh '''#!/bin/sh
                                    shopt -s nullglob
                                    set -e
                                    cd ${WORKSPACE}/jenkins/c7n_scripts/
                                    python3.6 ./get_temp_aws_creds.py -e $ENV
                                    '''
                                }
                                else{
                                    sh '''#!/bin/sh
                                    shopt -s nullglob
                                    set -e

                                    cd ${WORKSPACE}/jenkins/c7n_scripts/
                                    echo "creating accounts.yml"
                                    python3.6 ./create-accounts.py $ENV . --acct '''+params.ProjectId+'''
                                    cat ./accounts.yml
                                    echo "creating accounts_global.yml"
                                    python3.6 ./create-accounts.py $ENV . --globalservice --acct '''+params.ProjectId+'''

                                    python3.6 ./get_temp_aws_creds.py -e $ENV
                                    '''
                                }
                            }
                        }
                    }
                }
                stage('SLS Deploy') {
                    steps{
                        echo "SLS: deploy to a single account"
                        script{
                            ansiColor('xterm') {
                                def slsWorkDir = "${WORKSPACE}/serverless/${params.SLS_APPS_CHANGED}/"
                                pipInstall(slsWorkDir,"requirements.txt")


                                accounts_to_deploy_Yaml = readYaml(file: '''./serverless/'''+SLS_APPS_CHANGED+'''/deployment_config.yml''')
                                accounts_json_list = accounts_to_deploy_Yaml[ENV]["deployment_accounts"]
                                if(accounts_json_list[0].name == 'all') {
                                    // make the accounts.yml
                                    sh '''#!/bin/sh
                                    shopt -s nullglob
                                    set -e

                                    cd ${WORKSPACE}/jenkins/c7n_scripts/
                                    echo "creating accounts.yml"
                                    python3.6 ./create-accounts.py $ENV .
                                    cat ./accounts.yml
                                    '''

                                    // grab the accounts.yml
                                    valuesYaml = readYaml (file: './jenkins/c7n_scripts/accounts.yml')
                                    accounts_json_list = valuesYaml."accounts"
                                }

                                account = accounts_json_list[0]
                                account_id = account.name
                                println('will deploy to: ' + account_id)

                                deploymentbucket = account.name+'-lambda-package-'+account.regions[0]
                                println('DeploymentBucket: '+deploymentbucket )
                                if(accounts_json_list.size() > 1) {
                                    SLS_MULTIACCOUNT = true
                                }else{
                                    SLS_MULTIACCOUNT = false
                                }


                                sh '''#!/bin/sh
                                shopt -s nullglob
                                set -e
                                . ~/.nvm/nvm.sh
                                nvm use --lts
                                
                                cd ${WORKSPACE}/jenkins/c7n_scripts/
                                python3.6 ./get_temp_aws_creds.py -e $ENV -a '''+account_id+'''

                                cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/

                                mkdir -p ./config
                                cp ${WORKSPACE}/ext_config/*.pem ./config

                                mkdir -p ./shared_config
                                cp ${WORKSPACE}/shared_config/* ./shared_config

                                if [[ -f ./config/boot_config.ini ]]
                                then
                                    if [[ $ENV == pre-merge || $ENV == post-merge ]]
                                    then
                                        sed -i "s/ENV/dev/g" ./config/boot_config.ini
                                    else
                                        sed -i "s/ENV/$ENV/g" ./config/boot_config.ini
                                    fi
                                fi

                                npm install
                                sls deploy -s $ENV --deploymentbucket '''+deploymentbucket+'''
                                '''
                            }
                        }
                    }
                }
                stage('SLS BDD') {
                    when {
                        expression { return params.RUN_BEHAVE_TEST }
                    }
                    steps{
                        echo "SLS: Run BDD Testing"
                        ansiColor('xterm') {
                            sh '''#!/bin/sh
                            shopt -s nullglob
                            set -e

                            cd ${WORKSPACE}/jenkins/c7n_scripts
                            sh ./update-boot-config-sls.sh ${WORKSPACE} $SLS_APPS_CHANGED

                            cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/lambda
                            echo `pwd`
                            for i in *; do
                                echo 'i: ' $i
                                 if [[ $i != 'util' && $i != '__pycache__' ]]
                                then
                                    if [[ -d $i ]]
                                    then
                                        if [[ -d $i/tests ]]
                                        then
                                            cd ./$i/tests
                                            echo `pwd`
                                            behave
                                            cd ${WORKSPACE}/jenkins/c7n_scripts/
                                            python3.6 ./get_temp_aws_creds.py -e $ENV
                                            cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/lambda
                                        else
                                            echo "${WORKSPACE}/serverless/$SLS_APPS_CHANGED/lambda/$i/tests/ not present. Exiting with failure"
                                            exit 1
                                        fi
                                    fi
                                fi
                            done
                            if [[ -d ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/stepfunctions ]]
                            then
                                cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/stepfunctions
                                for i in *; do
                                if [[ $i != 'util' ]]
                                then
                                    if [[ -d $i ]]
                                    then
                                        if [[ -d $i/tests ]]
                                        then
                                            cd ./$i/tests
                                            echo 'Step Functions Behave Testing'
                                            behave
                                            cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/stepfunctions
                                        else
                                            echo "${WORKSPACE}/serverless/$SLS_APPS_CHANGED/stepfunctions/$i/tests/ not present. Exiting with failure"
                                            exit 1
                                        fi
                                    fi
                                fi
                                done
                            fi
                            cd ${WORKSPACE}
                            '''
                            findPattern(params.BDD_FEATURE_REGEX, false, true)
                            findPattern(params.BDD_SCENARIO_REGEX, false, true)
                            findPattern(params.BDD_STEP_REGEX, false, true)
                        }
                    }
                }
                /**
                stage('SLS Multi Account') {
                    when {
                        expression {return SLS_MULTIACCOUNT}
                    }
                    steps{
                        echo "SLS: deploy multiple accounts"
                        script{
                            ansiColor('xterm') {
                                 // println(SLS_APPS_CHANGED)
                                accounts_to_deploy_Yaml = readYaml(file: '''./serverless/'''+SLS_APPS_CHANGED+'''/deployment_config.yml''')
                                // println('''accounts to deploy: '''+accounts_to_deploy_Yaml)
                                accounts_json_list = accounts_to_deploy_Yaml.ENV."deployment_accounts"

                                if(accounts_json_list[0].name == 'all') {
                                    // make the accounts.yml
                                    sh '''#!/bin/sh
                                    shopt -s nullglob
                                    set -e

                                    cd ${WORKSPACE}/jenkins/c7n_scripts/
                                    echo "creating accounts.yml"
                                    python3.6 ./create-accounts.py $ENV .
                                    cat ./accounts.yml
                                    '''

                                    // grab the accounts.yml
                                    valuesYaml = readYaml (file: './jenkins/c7n_scripts/accounts.yml')
                                    accounts_json_list = valuesYaml."accounts"
                                }
                                println('''accounts_json_list'''+accounts_json_list)

                                //cannot do for loop. Variables get assigned wrong.
                                def tasks = [:]
                                for(def i = 1; i < accounts_json_list.size(); i++) {
                                    account = accounts_json_list[i]

                                    println('''account beginning loop: ''' + account)
                                    // tasks['''task_'''+account.name] = {
                                        // get credentials for that account
                                        // sls deploy like below
                                        println('''account inside task: ''' + account)
                                        deploymentbucket = account.name+'-lambda-package-'+account.regions[0]
                                        println('DeploymentBucket: '+deploymentbucket )
                                        println('account name: '+account.name )
                                        println('region: '+account.regions[0] )

                                        sh '''#!/bin/sh
                                        shopt -s nullglob
                                        set -e
                                        . ~/.nvm/nvm.sh
                                        nvm use --lts

                                        cd ${WORKSPACE}/jenkins/c7n_scripts/
                                        python3.6 get_temp_aws_creds.py -a '''+account.name+'''



                                        cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/
                                        sls deploy -s $ENV --deploymentbucket '''+deploymentbucket+'''
                                        '''
                                        //}
                                }
                                // println(tasks)
                                // parallel tasks
                            }
                        }
                    }
                }
                **/
            }
        }

/**
        stage('compliance-dashboard'){
            when {
                expression { return params.IS_DASHBOARD_CHANGE }
            }
            stages{
                // stage('SLS Run PyLint'){
                //     when {
                //         expression { return params.RUN_LINT }
                //     }
                //     steps{
                //         echo "SLS: Running PyLint"
                //         ansiColor('xterm') {
                //             sh '''#!/bin/sh
                //             shopt -s nullglob
                //             set -e

                //             cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED
                //             find . -type f -name "*.py" | xargs pylint --rcfile=${WORKSPACE}/pylintrc.cfg . > pylint.log || true
                //             cat pylint.log
                //             echo "Finished Pylint execution"
                //             '''
                //             findPattern(params.PYLINT_REGEX, true, params.RUN_LINT)
                //         }
                //     }
                // }
                // stage('SLS Run Unit Test'){
                //     when {
                //         expression { return params.RUN_UNIT_TEST }
                //     }
                //     steps{
                //         echo "SLS: Running Unit Test"
                //         script{
                //             ansiColor('xterm') {
                //                 def slsWorkDir = "${WORKSPACE}/serverless/${params.SLS_APPS_CHANGED}/"
                //                 pipInstall(slsWorkDir,"requirements.txt")

                //                 sh '''#!/bin/sh
                //                 shopt -s nullglob
                //                 set -e

                //                 cd ${WORKSPACE}/jenkins/c7n_scripts
                //                 sh ./update-boot-config-sls.sh ${WORKSPACE} $SLS_APPS_CHANGED

                //                 cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/glue
                //                 pwd
                //                 ls -la
                //                 for i in *; do
                //                     if [[ -d $i ]]
                //                     then
                //                         echo "$i is a directory"
                //                         cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/glue/$i/tests
                //                         pytest
                //                         # python3.6 -m unittest
                //                     fi
                //                 done
                //                 '''
                //                 findPattern(params.UNIT_TEST_REGEX, true, params.RUN_UNIT_TEST)
                //             }
                //         }
                //     }
                // }
                stage('SLS get credentials'){
                    steps{
                        echo "SLS: getting credentials"
                        script{
                            ansiColor('xterm'){
                                sh '''#!/bin/sh
                                shopt -s nullglob
                                set -e
                                cd ${WORKSPACE}/jenkins/c7n_scripts/
                                python3.6 ./get_temp_aws_creds.py -e $ENV
                                '''
                            }
                        }
                    }
                }
                stage('SLS Deploy'){
                    steps{
                        echo "SLS: deploy dashboard backend"
                        script{
                            ansiColor('xterm') {
                                def slsWorkDir = "${WORKSPACE}/serverless/${params.SLS_APPS_CHANGED}/"
                                pipInstall(slsWorkDir,"requirements.txt")

                                accounts_to_deploy_Yaml = readYaml(file: '''./serverless/'''+SLS_APPS_CHANGED+'''/deployment_config.yml''')
                                accounts_json_list = accounts_to_deploy_Yaml[ENV]["deployment_accounts"]
                                if(accounts_json_list[0].name == 'all'){
                                    // make the accounts.yml
                                    sh '''#!/bin/sh
                                    shopt -s nullglob
                                    set -e

                                    cd ${WORKSPACE}/jenkins/c7n_scripts/
                                    echo "creating accounts.yml"
                                    python3.6 ./create-accounts.py $ENV .
                                    cat ./accounts.yml
                                    '''

                                    // grab the accounts.yml
                                    valuesYaml = readYaml (file: './jenkins/c7n_scripts/accounts.yml')
                                    accounts_json_list = valuesYaml."accounts"
                                }

                                account = accounts_json_list[0]
                                account_id = account.name
                                println('will deploy to: ' + account_id)

                                deploymentbucket = account.name+'-lambda-package-'+account.regions[0]
                                scriptbucket = account.bucket
                                logs_bucket = scriptbucket.split('/')[0]
                                println('DeploymentBucket: '+deploymentbucket )
                                println('LogsBucket: '+logs_bucket )
                                sh '''#!/bin/sh
                                shopt -s nullglob
                                set -e
                                . ~/.nvm/nvm.sh
                                nvm use --lts
                                cd ${WORKSPACE}/jenkins/c7n_scripts/
                                python3.6 ./get_temp_aws_creds.py -e $ENV -a '''+account_id+'''

                                cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/glue/scripts

                                for f in *.py
                                    do
                                        sed -i "s/LOGS_BUCKET/'''+logs_bucket+'''/g" ./$f
                                        aws s3 cp $f s3://'''+scriptbucket+'''/
                                    done

                                sls deploy -s $ENV --deploymentbucket '''+deploymentbucket+'''
                                '''
                            }
                        }
                    }
                }
                // stage('SLS BDD'){
                //     when {
                //         expression { return params.RUN_BEHAVE_TEST }
                //     }
                //     steps{
                //         echo "SLS: Run BDD Testing"
                //         ansiColor('xterm') {
                //             sh '''#!/bin/sh
                //             shopt -s nullglob
                //             set -e

                //             cd ${WORKSPACE}/jenkins/c7n_scripts
                //             sh ./update-boot-config-sls.sh ${WORKSPACE} $SLS_APPS_CHANGED

                //             cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/lambda
                //             echo `pwd`
                //             for i in *; do
                //                 echo 'i: ' $i
                //                  if [[ $i != 'util' && $i != '__pycache__' ]]
                //                 then
                //                     if [[ -d $i ]]
                //                     then
                //                         if [[ -d $i/tests ]]
                //                         then
                //                             cd ./$i/tests
                //                             echo `pwd`
                //                             behave
                //                             cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/lambda
                //                         else
                //                             echo "${WORKSPACE}/serverless/$SLS_APPS_CHANGED/lambda/$i/tests/ not present. Exiting with failure"
                //                             exit 1
                //                         fi
                //                     fi
                //                 fi
                //             done
                //             if [[ -d ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/stepfunctions ]]
                //             then
                //                 cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/stepfunctions
                //                 for i in *; do
                //                 if [[ $i != 'util' ]]
                //                 then
                //                     if [[ -d $i ]]
                //                     then
                //                         if [[ -d $i/tests ]]
                //                         then
                //                             cd ./$i/tests
                //                             echo 'Step Functions Behave Testing'
                //                             behave
                //                             cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/stepfunctions
                //                         else
                //                             echo "${WORKSPACE}/serverless/$SLS_APPS_CHANGED/stepfunctions/$i/tests/ not present. Exiting with failure"
                //                             exit 1
                //                         fi
                //                     fi
                //                 fi
                //                 done
                //             fi
                //             cd ${WORKSPACE}
                //             '''
                //             findPattern(params.BDD_FEATURE_REGEX, false, true)
                //             findPattern(params.BDD_SCENARIO_REGEX, false, true)
                //             findPattern(params.BDD_STEP_REGEX, false, true)
                //         }
                //     }
                // }
                // stage('SLS Multi Account'){
                //     when {
                //         expression {return SLS_MULTIACCOUNT}
                //     }
                //     steps{
                //         echo "SLS: deploy multiple accounts"
                //         script{
                //             ansiColor('xterm'){
                //                  // println(SLS_APPS_CHANGED)
                //                 accounts_to_deploy_Yaml = readYaml(file: '''./serverless/'''+SLS_APPS_CHANGED+'''/deployment_config.yml''')
                //                 // println('''accounts to deploy: '''+accounts_to_deploy_Yaml)
                //                 accounts_json_list = accounts_to_deploy_Yaml.ENV."deployment_accounts"

                //                 if(accounts_json_list[0].name == 'all'){
                //                     // make the accounts.yml
                //                     sh '''#!/bin/sh
                //                     shopt -s nullglob
                //                     set -e

                //                     cd ${WORKSPACE}/jenkins/c7n_scripts/
                //                     echo "creating accounts.yml"
                //                     python3.6 ./create-accounts.py $ENV .
                //                     cat ./accounts.yml
                //                     '''

                //                     // grab the accounts.yml
                //                     valuesYaml = readYaml (file: './jenkins/c7n_scripts/accounts.yml')
                //                     accounts_json_list = valuesYaml."accounts"
                //                 }
                //                 println('''accounts_json_list'''+accounts_json_list)

                //                 //cannot do for loop. Variables get assigned wrong.
                //                 def tasks = [:]
                //                 for(def i = 1; i < accounts_json_list.size(); i++){
                //                     account = accounts_json_list[i]

                //                     println('''account beginning loop: ''' + account)
                //                     // tasks['''task_'''+account.name] = {
                //                         // get credentials for that account
                //                         // sls deploy like below
                //                         println('''account inside task: ''' + account)
                //                         deploymentbucket = account.name+'-lambda-package-'+account.regions[0]
                //                         println('DeploymentBucket: '+deploymentbucket )
                //                         println('account name: '+account.name )
                //                         println('region: '+account.regions[0] )

                //                         sh '''#!/bin/sh
                //                         shopt -s nullglob
                //                         set -e
                //                         . ~/.nvm/nvm.sh
                //                         nvm use --lts

                //                         cd ${WORKSPACE}/jenkins/c7n_scripts/
                //                         python3.6 get_temp_aws_creds.py -a '''+account.name+'''



                //                         cd ${WORKSPACE}/serverless/$SLS_APPS_CHANGED/
                //                         sls deploy -s $ENV --deploymentbucket '''+deploymentbucket+'''
                //                         '''
                //                         //}
                //                 }
                //                 // println(tasks)
                //                 // parallel tasks
                //             }
                //         }
                //     }
                // }
            }
        }
**/
        stage('Amplify'){
            when {
                expression { return params.IS_AMPLIFY_CHANGE || params.IS_QUALMAP_CHANGE }
            }
            stages{
                stage('Deploy Amplify') {
                    steps{
                        ansiColor('xterm') {
                            println('deploying amplify backend')
                            sh '''#!/bin/sh
                            shopt -s nullglob
                            set -e
                            . ~/.nvm/nvm.sh
                            nvm use --lts

                            cd ${WORKSPACE}/jenkins/c7n_scripts/
                            python3.6 get_temp_aws_creds.py -e $ENV
                            cat ~/.aws/credentials

                            cd ${WORKSPACE}/qualmap/webapp
                            npm install
                            ./amplify_headless_init.sh $ENV
                            amplify publish -c --yes
                            '''
                       }
                    }
                }
                stage('Qualmap Deploy') {
                    when {
                        expression { return params.IS_QUALMAP_CHANGE }
                    }
                    steps{
                        ansiColor('xterm') {
                            sh'''
                            #!/bin/sh
                            shopt -s nullglob
                            set -e
                            . ~/.nvm/nvm.sh
                            nvm use --lts
                            
                            cd ${WORKSPACE}/jenkins/c7n_scripts/
                            python3.6 get_temp_aws_creds.py -e $ENV
                            cat ~/.aws/credentials

                            cd ${WORKSPACE}/qualmap/webapp

                            amplify publish -c --yes

                            cat ./amplify/team-provider-info.json
                            '''
                        }
                    }
                }
                stage('Qualmap Test') {
                    when {
                        expression { return params.IS_QUALMAP_CHANGE && params.RUN_UNIT_TEST}
                    }
                    steps{
                        ansiColor('xterm') {
                            sh'''
                            #!/bin/sh
                            shopt -s nullglob
                            set -e
                            . ~/.nvm/nvm.sh
                            nvm use --lts
                            
                            cd ${WORKSPACE}/jenkins/c7n_scripts/
                            python3.6 get_temp_aws_creds.py -e $ENV
                            cat ~/.aws/credentials

                            cd ${WORKSPACE}/qualmap/webapp
                            npm run test
                            '''
                            findPattern(params.UNIT_TEST_REGEX, true, params.RUN_UNIT_TEST)
                        }
                    }
                }
                stage('Deploy Secrets') {
                    steps{
                        ansiColor('xterm') {
                            println('deploying secrets to secrets manager')
                            sh '''
                            #!/bin/sh
                            cd ${WORKSPACE}/jenkins/c7n_scripts/
                            python3.6 get_temp_aws_creds.py -e $ENV
                            cat ~/.aws/credentials

                            python3.6 upload-to-ssm.py --cfnstack amplify-jnjqualmap
                            '''
                        }
                    }
                }
            }
        }
        stage('SCP') {
            when {
                expression { return params.IS_C7N_SCP_CHANGE }
            }
            stages{
                stage('SCP Deploy') {
                    steps{
                        ansiColor('xterm') {
                            println('deploying scp to orgs')
                            sh '''#!/bin/sh
                            shopt -s nullglob
                            set -e

                            cd ${WORKSPACE}/jenkins/c7n_scripts
                            pipenv run python ./deploy-scps.py -e $ENV
                            '''
                        }
                    }
                }
                stage('SCP BDD') {
                    when {
                        expression { return params.RUN_BEHAVE_TEST }
                    }
                    steps{
                        ansiColor('xterm') {
                    sh '''#!/bin/sh
                    shopt -s nullglob
                    set -e

                    if [[ -d ${WORKSPACE}/custodian/scps/bdd ]]
                    then
                        cd ${WORKSPACE}/custodian/scps/bdd
                        behave
                    else
                        echo "custodian/scps/bdd not present. Exiting with failure"
                        exit 1
                    fi
                    '''
                        }
                    findPattern(params.BDD_FEATURE_REGEX, false, true)
                    findPattern(params.BDD_SCENARIO_REGEX, false, true)
                    findPattern(params.BDD_STEP_REGEX, false, true)
                    }
                }
            }
        }

        stage('Accounts IAM Policy'){
            when {
                expression { return params.IS_ACCOUNTS_IAM_CHANGE }
            }
            stages{
                stage('Upload IAM Policy'){
                    steps{
                        script{
                            valuesYaml = readYaml (file: './jenkins/deployment_targets.yaml')
                            config = valuesYaml."aws"."${params.ENV}"
                            iam_policy_bucket = config["iam_policy_bucket"]

                            sh '''#!/bin/bash
                            set -e

                            if [[ -d ${WORKSPACE}/aws_accounts ]]
                            then
                                cd ${WORKSPACE}/aws_accounts
                                for f in *.y*ml
                                do
                                    aws s3 cp $f s3://'''+iam_policy_bucket+'''/ --sse
                                done
                            fi
                            '''
                        }
                    }
                }
                stage('BDD'){
                    when {
                        expression { return params.RUN_BEHAVE_TEST }
                    }
                    steps{
                        sh '''#!/bin/bash
                        set -e

                        if [[ -d ${WORKSPACE}/aws_accounts ]]
                        then
                            cd ${WORKSPACE}/aws_accounts/bdd
                            behave
                        fi
                        '''
                        findPattern(params.BDD_FEATURE_REGEX, false, true)
                        findPattern(params.BDD_SCENARIO_REGEX, false, true)
                        findPattern(params.BDD_STEP_REGEX, false, true)
                    }
                }
            }
        }

        /**
        stage('C7N Run All BDD') {
            when {
                expression { return params.RUN_ALL_C7N_BEHAVE_TESTS }
            }
            steps{
                script{
                    ansiColor('xterm') {
                        println('need to run all bdd')
                        sh '''#!/bin/sh
                        shopt -s nullglob
                        set -e

                        cd ${WORKSPACE}/custodian/policies/
                        for resource in *
                        do
                            if [[ -d $resource && $resource != "bdd_utils" ]]
                            then
                                cd $resource
                                if [[ -d ./bdd ]]
                                then
                                    cd bdd
                                    echo "current directory:"
                                    echo `pwd`
                                    behave
                                    cd ${WORKSPACE}/custodian/policies/
                                else
                                    echo "custodian/policies/$resource/bdd not present. Exiting with failure"
                                    exit 1
                                fi
                            fi
                        done
                        '''
                        findPattern(params.BDD_FEATURE_REGEX, false, true)
                        findPattern(params.BDD_SCENARIO_REGEX, false, true)
                        findPattern(params.BDD_STEP_REGEX, false, true)
                    }
                }
            }
        }
        **/
        // stage('Nextbot Tag Resources') {
        //     steps{
        //         script{
        //             ansiColor('xterm') {
        //                 sh '''#!/bin/sh
        //                 shopt -s nullglob
        //                 set -e

        //                 cd ${WORKSPACE}/jenkins/c7n_scripts/
        //                 echo "creating accounts.yml"
        //                 python3.6 ./create-accounts.py $ENV .
        //                 cat ./accounts.yml
        //                 '''

        //                 valuesYaml = readYaml (file: './jenkins/deployment_targets.yaml')
        //                 config = valuesYaml."aws"."${params.ENV}"
        //                 central_account = config["central_account"]

        //                 valuesYamlAccounts = readYaml (file: './jenkins/c7n_scripts/accounts.yml')
        //                 println(valuesYamlAccounts)
        //                 accounts_json_list = valuesYamlAccounts."accounts"

        //                 for(account in accounts_json_list) {
        //                     if(account.name == central_account) {
        //                         sh'''
        //                         cd ${WORKSPACE}/jenkins/c7n_scripts/
        //                         echo "Tagging '''+account.name+'''"
        //                         python3.6 ./nextbot_tagging.py --projectid '''+account.name+''' --centralaccount
        //                         '''
        //                     }
        //                     else{
        //                         sh'''
        //                         cd ${WORKSPACE}/jenkins/c7n_scripts/
        //                         echo "Tagging '''+account.name+'''"
        //                         python3.6 ./nextbot_tagging.py --projectid '''+account.name+'''
        //                         '''
        //                     }
        //                 }
        //             }
        //         }
        //     }
        // }
    }

    post {
        failure {
            script {
                currentBuild.result = "FAILED"
                step([$class: 'StashNotifier'])
            }
        }
        success {
            script {
                currentBuild.result = "SUCCESS"
                step([$class: 'StashNotifier'])
            }
        }
    }
}

def info(String message) {
  print "\033[1;36m[Info] $message\033[0m "
}

def success(String message) {
  print "\033[1;32m[Success] $message\033[0m "
}

def error(String message) {
  print "\033[1;31m[Error] $message \033[0m "
}

def pipInstall(WorkDir, requirementsFile) {
    sh'''#!/bin/sh
    shopt -s nullglob
    set -e

    WORKDIR="'''+WorkDir+'''"
    REQUIREMENTSFILE="'''+requirementsFile+'''"

    cd ${WORKDIR}

    if [[ -f Pipfile ]]
    then
        pipenv run pip freeze > requirements.txt
    fi
    python3.6 -m pip install \
    --user \
    --pre \
    --extra-index-url=http://itsusralsp07062.jnj.com:8090 \
    --trusted-host=itsusralsp07062.jnj.com \
    --extra-index-url https://pypi.jjapi.jnj.com/v1/itx-abp/private/${ENV}/ \
    --extra-index-url https://pypi.jjapi.jnj.com/v1/itx-alz/shared/production/ \
    -r $REQUIREMENTSFILE
    '''
}

def findPattern(String regex, Boolean failIfFound, Boolean enabled) {
  if (enabled) {
    info "find pattern Started with regex=> ${regex}"
    def logs = currentBuild.rawBuild.getLog(10000).join('\n')
    //info "logs: " + logs
    //echo "$logs"
    Pattern pattern = Pattern.compile(regex)
    Matcher matcher = pattern.matcher(logs)
    // Check all occurrences
    if (matcher.find()) {
      // got a match to the regex
      if (failIfFound) {
        // got a match and we should fail if match found
        error "got a match and we should fail if match found"
        currentBuild.result = 'FAILED'
      } else {
        // got a match and we should not fail if a match found
        success "got a match and we should not fail if a match found, hence verification passed"
      }
    } else {
      // no match found
      if (!failIfFound) {
        //no match found, but this pattern must be present, hence fail it
        //info "thelogs: $logs"
        error "no match found in the logs, but this pattern must be present, hence fail it"
        currentBuild.result = 'FAILED'
      } else {
        //no match found and this pattern should not be present, hence pass
        success "no match found and this pattern should not be present, hence verification passed"
      }
    }
  } else {
    info "Pattern Match Disabled"
  }
}

def listDirs(dir) {
                def output = sh returnStdout: true, script: "ls -l ${dir} | grep ^d | awk '{print \$9}'"
    println("listDirs output: ")
    println(output)
    dlist = output.tokenize('\n').collect() { it }
    return dlist
}

def deployC7nOnEC2(String custodianBucket, String workdir, String deploy_list) {
	sshagent(credentials: ["${params.SSH_CREDENTIAL_ID}"]) {
		sh "ssh -o StrictHostKeyChecking=no \
		${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER} \
		\"nice -n 20 ${params.APP_DEPLOY_DIR}/${workDir}/jenkins/c7n_scripts/run_c7n_deploy.sh \
		${deploy_list} \
		${params.APP_DEPLOY_DIR}/${workDir}/custodian/policies \
		${params.APP_DEPLOY_DIR}/${workDir} \
		${params.ENV} \
        ${params.USER_NAME} \
        ${custodianBucket} \
        \""
	}
}

def installCustodianPipEnvOnEc2(String custodianDir, String env) {
	sshagent(credentials: ["${params.SSH_CREDENTIAL_ID}"]) {
		sh "ssh -o StrictHostKeyChecking=no \
		${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER} \
		\"cd ${custodianDir} && \
        export sddcapi_boot_dir=\"/app/xbot/deploy/aws-nextBot-pipeline/ext_config\" && \
        python3.6 ~/.local/bin/pipenv --bare install && \
        python3.6 ~/.local/bin/pipenv run python utils/inject_c7n_classes.py && \
        python3.6 ~/.local/bin/pipenv --bare install --dev && \
        python3.6 ~/.local/bin/pipenv run python -m pip install --upgrade pip==20.0.1 && \
        python3.6 ~/.local/bin/pipenv run python -m pip install --extra-index-url=http://itsusralsp07062.jnj.com:8090 --trusted-host=itsusralsp07062.jnj.com --extra-index-url https://pypi.jjapi.jnj.com/v1/itx-abp/private/${env}/ --extra-index-url https://pypi.jjapi.jnj.com/v1/itx-alz/shared/production/ -r ../jenkins/requirements-jnj.txt && \
        python3.6 ~/.local/bin/pipenv run python -m pip install \"boto3>=1.17.54\"\
        \""
	}
}

def installCustodianDeployScriptPipEnvOnEc2(String custodianScriptDir) {
	sshagent(credentials: ["${params.SSH_CREDENTIAL_ID}"]) {
		sh "ssh -o StrictHostKeyChecking=no \
		${params.USER_NAME}@${params.C7N_EC2_DEPLOY_SERVER} \
		\"cd ${custodianScriptDir} && \
        python3.6 ~/.local/bin/pipenv install \
        \""
	}
}

def installCustodianPipEnv() {
    sh '''#!/bin/bash
    set -e

    cd ${WORKSPACE}/custodian/
    pipenv --bare install --dev

    pipenv run python -m pip install --upgrade pip==20.0.1
    pipenv run python -m pip install --extra-index-url=http://itsusralsp07062.jnj.com:8090 --trusted-host=itsusralsp07062.jnj.com --extra-index-url https://pypi.jjapi.jnj.com/v1/itx-abp/private/${env}/ --extra-index-url https://pypi.jjapi.jnj.com/v1/itx-alz/shared/production/ -r ../jenkins/requirements-jnj.txt
    pipenv run python -m pip install "boto3>=1.17.54"
    echo "Beginning class injection"
    pipenv run python ${WORKSPACE}/custodian/utils/inject_c7n_classes.py
    echo "Injection complete"
    '''
}

def installCustodianDeployScriptPipEnv() {
    sh '''#!/bin/bash
    set -e

    cd ${WORKSPACE}/jenkins/c7n_scripts
    pipenv --bare install
    '''  
}

def deployC7nOnAgent(String custodianBucket, String deploy_list) {
	sh '''#!/bin/sh
	set -e
	
	. ${WORKSPACE}/jenkins/c7n_scripts/run_c7n_deploy.sh \
    ''' + deploy_list + ''' \
	${WORKSPACE}/custodian/policies/ \
	${WORKSPACE} \
	${ENV} \
    jenkins \
    ''' + custodianBucket + ''' \
	'''
}
