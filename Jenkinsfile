#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH

    def toolbelt = tool 'toolbelt'

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Create Scratch Org') {

            rc = sh returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            if (rc != 0) { error 'hub org authorization failed' }

            // need to pull out assigned username
            rmsg = sh returnStdout: true, script: "\"${toolbelt}\" force:org:create -f config/developerOrg-scratch-def.json --json -s -a df13@makepositive.com"
            println(rmsg)
            def jsonSlurper = new JsonSlurperClassic()
            def robj = jsonSlurper.parseText(rmsg)
            if (robj.status != 0) { error 'org creation failed: ' + robj.message }
            SFDC_USERNAME=robj.username
            robj = null
        }

        stage('Create password for scratch org') {
			rc = sh returnStatus: true, script: "\"${toolbelt}\" force:user:password:generate --targetusername ${SFDC_USERNAME}"
            if (rc != 0) {
                error 'password generation failed'
            }
        }
		
        stage('Push To Test Org') {
            rc = sh returnStatus: true, script: "\"${toolbelt}\" force:source:push --targetusername ${SFDC_USERNAME}"
            if (rc != 0) {
                error 'push failed'
            }
            // assign permset
            rc = sh returnStatus: true, script: "\"${toolbelt}\" force:user:permset:assign --targetusername ${SFDC_USERNAME} --permsetname DreamHouse"
            if (rc != 0) {
                error 'permset:assign failed'
            }
        }
        
        //stage('Create Users in scratch org') {
		//	rc = sh returnStatus: true, script: "\"${toolbelt}\" force:user:create -a scratchOrg2@user2.com -f config/user-scratch-def.json --json --targetusername ${SFDC_USERNAME}"
        //    if (rc != 0) {
        //        error 'User creation failed'
        //    }
        //    
        //    rc = sh returnStatus: true, script: "\"${toolbelt}\" force:user:create -a scratchOrg2@user2.com -f config/user-scratch-def.json --json profileName="Chatter Free User" --targetusername ${SFDC_USERNAME}"
        //    if (rc != 0) {
        //        error 'User creation failed'
        //    }
        //}

        //stage('Run Provar test cases') {
		//	rc = sh returnStatus: true, script: "ant -f build.xml -DadminUser=${SFDC_USERNAME}"
        //    if (rc != 0) {
        //        error 'User creation failed'
        //    }
        //}

        //stage('Run Apex Test') {
        //    sh "mkdir -p ${RUN_ARTIFACT_DIR}"
        //    timeout(time: 120, unit: 'SECONDS') {
        //        rc = sh returnStatus: true, script: "\"${toolbelt}\"/sfdx force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --targetusername ${SFDC_USERNAME}"
        //        if (rc != 0) {
        //            error 'apex test run failed'
        //        }
        //    }
        //}

        stage('collect results') {
            junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
        }
    }
}
