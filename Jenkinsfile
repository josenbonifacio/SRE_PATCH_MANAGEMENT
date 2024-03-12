pipeline {
    agent any

    parameters {
        activeChoice(
            description: 'Select the maintenance group',
            name: 'INFO_MAINTENANCES',
            script: [
                $class: 'GroovyScript',
                fallbackScript: [
                    classpath: [],
                    script: 'return ["Error: Script Not Available"]'
                ],
                script: [
                    classpath: [],
                    sandbox: true,
                    script: '''
                        

                        import groovy.json.JsonSlurper
                        import java.net.HttpURLConnection
                        import java.net.URL
                        import java.io.OutputStream

                        //VARIABLES
                        def api_token
                        API_URL="http://172.18.0.1:8081/api_jsonrpc.php";

                        //JSONS BODY TOKEN
                        GET_TOKEN_PARAMS="{\\"jsonrpc\\":\\"2.0\\",\\"method\\":\\"user.login\\",\\"params\\":{\\"user\\":\\"Admin\\",\\"password\\":\\"zabbix\\"},\\"id\\":1,\\"auth\\":null}";

                        //CONNECTION & GLOBAL PARAMETERS
                        def con = new URL(API_URL).openConnection() as HttpURLConnection

                        //REQUEST TOKEN PARAMETERS
                        con.setRequestMethod("GET");
                        con.setRequestProperty("Content-Type", "application/json-rpc");
                        con.setDoOutput(true);
                        OutputStream os = con.getOutputStream();
                        os.write(GET_TOKEN_PARAMS.getBytes());
                        os.flush();
                        os.close();

                        if (con.getResponseCode() == 200) { // success
                            api_token = new JsonSlurper().parseText(con.getInputStream().getText('UTF-8'))
                        } else {
                            System.out.println("GET request did not work.");
                        }
                        con.disconnect();

                        // Return the token
                        return api_token
                    '''
                ]
            ]
        )

    }

    stages {


        stage('Example') {
            steps {
                script {
                    echo "Selected maintenance group: ${params.INFO_MAINTENANCES}"
                }
            }
        }
    }
}
