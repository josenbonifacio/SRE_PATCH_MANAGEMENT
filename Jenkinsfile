pipeline {
  agent any

  stages {
    stage('Parameters') {
      steps {
        script {
          properties([
            parameters([
              [$class: 'ChoiceParameter',
                choiceType: 'PT_SINGLE_SELECT',
                description: 'Select the Maintenance to see the details',
                filterLength: 1,
                filterable: false,
                name: 'INFO_MAINTENANCES',
                script: [
                  $class: 'GroovyScript',
                  fallbackScript: [
                    classpath: [],
                    sandbox: false,
                    script:
                    'return ["Error: Script Not Available"]'
                  ],
                  script: [
                    classpath: [],
                    sandbox: false,
                    script: '''
                    package maintenance_list

                    import groovy.json.JsonSlurper
                    import java.net.HttpURLConnection
                    import java.net.URL
                    import java.io.OutputStream

                    //VARIABLES
                    def api_token
                    API_URL = "http://172.18.0.1:8081/api_jsonrpc.php";

                    //JSONS BODY TOKEN
                    GET_TOKEN_PARAMS = "{\\"
                    jsonrpc\\":\\"
                    2.0\\",\\"
                    method\\":\\"
                    user.login\\",\\"
                    params\\":{\\"
                    user\\":\\"
                    Admin\\",\\"
                    password\\":\\"
                    zabbix\\"},\\"
                    id\\":1,\\"
                    auth\\":null}";

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
                    //JSONS BODY MAINTENANCE LIST
                    GET_MAINTENANCE_LIST = "{\\"jsonrpc\\":\\"2.0\\",\\"method\\":\\"maintenance.get\\",\\"params\\":{\\"output\\":\\"extend\\",\\"selectGroups\\":\\"extend\\",\\"selectTimeperiods\\":\\"extend\\",\\"selectTags\\":\\"extend\\"},\\"auth\\":" + "\\"" + api_token.result + "\\"" + ",\\"id\\":\\"1\\"}"

                    //REQUEST MAINTENANCE PARAMETERS
                    con = new URL(API_URL).openConnection() as HttpURLConnection
                    con.setRequestMethod("GET");
                    con.setRequestProperty("Content-Type", "application/json-rpc");
                    con.setDoOutput(true);
                    os = con.getOutputStream();
                    os.write(GET_MAINTENANCE_LIST.getBytes());
                    os.flush();
                    os.close();

                    //PARSE REQUEST
                    def groupslistfinal = []
                    if (con.getResponseCode() == 200) {
                      groupslist = new JsonSlurper().parseText(con.getInputStream().getText('UTF-8'))
                      groupslist.result.each {
                        result ->
                          groupslistfinal.add(result.name)
                      }
                      groupslistfinal.eachWithIndex {
                        it,
                        i -> println "$i : $it"
                      }
                      return groupslistfinal.sort()
                    } else {
                      println("HTTP response error")
                      System.exit(0)
                    }

                    con.disconnect();

                    '''
                  ]
                ]
              ]
            ])
          ])

        }
      }
    }
  }
}
