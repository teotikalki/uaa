# On Acceptance

## UAAC curl command to create IDP in the default zone

    uaac curl -k --data '{"type":"saml","config":"{\"emailDomain\":null,\"additionalConfiguration\":null,\"providerDescription\":null,\"externalGroupsWhitelist\":[],\"attributeMappings\":{},\"addShadowUserOnLogin\":true,\"storeCustomAttributes\":false,\"metaDataLocation\":\"https://dnd.login.identity.cf-app.com/saml/idp/metadata\",\"idpEntityAlias\":\"dnd.login.identity.cf-app.com\",\"zoneId\":\"uaa\",\"nameID\":\"urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified\",\"assertionConsumerIndex\":0,\"metadataTrustCheck\":false,\"showSamlLink\":false,\"linkText\":\"Login with Local SAML IDP(dnd.login.identity.cf-app.com)\",\"iconUrl\":null,\"groupMappingMode\":\"EXPLICITLY_MAPPED\",\"skipSslValidation\":true,\"socketFactoryClassName\":null}","originKey":"dnd.login.identity.cf-app.com","name":"dnd.login.identity.cf-app.com","version":0,"created":1485293040218,"last_modified":1485293040218,"active":true,"identityZoneId":"uaa"}' /identity-providers -XPOST -H"Content-Type:application/json" -H"Accept:application/json"

## UAAC command to create client for grant

    uaac client add --name "Saml 2 Assertion Client" --scope "openid" --authorized_grant_types urn:ietf:params:oauth:grant-type:saml2-bearer --authorities uaa.resource --autoapprove true -s secret saml2client



## Token assertion

###  Generate an assertion using the following command

    ./gradlew mockAssertion -Dfile=./server/src/test/resources/saml/mock-assertion-acceptance-dnd.yml

###  Assign the output to a variable

    export ASSERTION=<paste value here>

###  Run the command

    curl -k -H"Accept: application/json" -d "client_id=saml2client" -d "client_secret=secret" -d "grant_type=urn:ietf:params:oauth:grant-type:saml2-bearer" -d "assertion=$ASSERTION" https://login.identity.cf-app.com/oauth/token/alias/login.identity.cf-app.com

# Same Steps but with Local UAA

## Before you start
*When we generate the assertion using the `gradlew mockAssertion` command
the gradle build script modifies the working directory, this causes the Tomcat container to undeploy the 
uaa application.*

If you are aware of how we can execute this task without running all the previous build segments, let us know!

###  Generate an assertion using the following command

    ./gradlew mockAssertion -Dfile=./server/src/test/resources/saml/mock-assertion-localhost-against-dnd.yml


## Start the UAA


    ./gradlew run
    
## There is a bug
https://www.pivotaltracker.com/story/show/138365807

    curl -v http://localhost:8080/uaa/saml/metadata

## Setup

    uaac target http://localhost:8080/uaa
    uaac token client get admin -s adminsecret
    
## UAAC curl command to create IDP in the default zone

    uaac curl -k --data '{"type":"saml","config":"{\"emailDomain\":null,\"additionalConfiguration\":null,\"providerDescription\":null,\"externalGroupsWhitelist\":[],\"attributeMappings\":{},\"addShadowUserOnLogin\":true,\"storeCustomAttributes\":false,\"metaDataLocation\":\"https://dnd.login.identity.cf-app.com/saml/idp/metadata\",\"idpEntityAlias\":\"dnd.login.identity.cf-app.com\",\"zoneId\":\"uaa\",\"nameID\":\"urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified\",\"assertionConsumerIndex\":0,\"metadataTrustCheck\":false,\"showSamlLink\":false,\"linkText\":\"Login with Local SAML IDP(dnd.login.identity.cf-app.com)\",\"iconUrl\":null,\"groupMappingMode\":\"EXPLICITLY_MAPPED\",\"skipSslValidation\":true,\"socketFactoryClassName\":null}","originKey":"dnd.login.identity.cf-app.com","name":"dnd.login.identity.cf-app.com","version":0,"created":1485293040218,"last_modified":1485293040218,"active":true,"identityZoneId":"uaa"}' /identity-providers -XPOST -H"Content-Type:application/json" -H"Accept:application/json"

## UAAC command to create client for grant

    uaac client add --name "Saml 2 Assertion Client" --scope "openid" --authorized_grant_types urn:ietf:params:oauth:grant-type:saml2-bearer --authorities uaa.resource --autoapprove true -s secret saml2client

## Token assertion

###  Assign the output to a variable

    export ASSERTION=`cat /tmp/assertion.txt`

###  Run the command

    curl -k -H"Accept: application/json" -d "client_id=saml2client" -d "client_secret=secret" -d "grant_type=urn:ietf:params:oauth:grant-type:saml2-bearer" -d "assertion=$ASSERTION" http://localhost:8080/uaa/oauth/token/alias/cloudfoundry-saml-login