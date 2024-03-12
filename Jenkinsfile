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
                    'return ["Error"]'
                  ],
                  script: [
                    classpath: [],
                    sandbox: false,
                    script: '''
                    import groovy.json.JsonSlurper
                    import java.net.HttpURLConnection
                    import java.net.URL
                    import java.io.OutputStream

                    //VARIABLES
                    def api_token
                    API_URL = "http://172.18.0.1:8081/api_jsonrpc.php";

                    //JSONS BODY TOKEN
                    GET_TOKEN_PARAMS = "{\\"jsonrpc\\":\\"2.0\\",\\"method\\":\\"user.login\\",\\"params\\":{\\"user\\":\\"Admin\\",\\"password\\":\\"zabbix\\"},\\"id\\":1,\\"auth\\":null}";

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
              ],
              [$class: 'DynamicReferenceParameter', 
                choiceType: 'ET_UNORDERED_LIST', 
                description: 'Details of Maintenance', 
                name: 'INFO_MAINTENANCE_DETAILS', 
                referencedParameters: 'INFO_MAINTENANCES', 
                script: [
                  $class: 'GroovyScript',
                  fallbackScript: [
                    classpath: [],
                    sandbox: false,
                    script:
                    'return ["Error"]'
                  ],
                  script: [
                    classpath: [],
                    sandbox: false,
                    script: '''

                    import groovy.json.JsonSlurper;
                    import java.util.concurrent.TimeUnit
                    
                    def api_token
                    API_URL="http://172.18.0.1:8081/api_jsonrpc.php";

                    //JSONS BODY TOKEN
                    GET_TOKEN_PARAMS = "{\\"jsonrpc\\":\\"2.0\\",\\"method\\":\\"user.login\\",\\"params\\":{\\"user\\":\\"Admin\\",\\"password\\":\\"zabbix\\"},\\"id\\":1,\\"auth\\":null}";
                    
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

                    //JSON BODY MAINTENANCE DETAILS
                    GET_MAINTENANCE_DETAILS="{\\"jsonrpc\\":\\"2.0\\",\\"method\\":\\"maintenance.get\\",\\"params\\":{\\"output\\":\\"extend\\",\\"selectGroups\\":\\"extend\\",\\"selectTimeperiods\\":\\"extend\\",\\"selectTags\\":\\"extend\\"},\\"auth\\":" + "\\"" + api_token.result + "\\"" + ",\\"id\\":\\"1\\"}"
                    con = new URL(API_URL).openConnection() as HttpURLConnection
                    con.setRequestMethod("GET");
                    con.setRequestProperty("Content-Type", "application/json-rpc");
                    con.setDoOutput(true);
                    os = con.getOutputStream();
                    os.write(GET_MAINTENANCE_DETAILS.getBytes());
                    os.flush();
                    os.close()

                    //PARSE REQUEST
                    def groupslistfinal = []
                    if (con.getResponseCode() == 200) {
                        groupslist = new JsonSlurper().parseText(con.getInputStream().getText('UTF-8'))
                        groupslist.result.each { result ->
                            if(result.name.equals(INFO_MAINTENANCES)){
                                groupslistfinal.add("Groups " + result.groups.name)
                                groupslistfinal.add("Active since - " + new java.text.SimpleDateFormat("MM/dd/yyyy HH:mm:ss").format(new Date(Integer.parseInt(result.active_since)*1000L)))
                                groupslistfinal.add("Active till - " + new java.text.SimpleDateFormat("MM/dd/yyyy HH:mm:ss").format(new Date(Integer.parseInt(result.active_till)*1000L)))
                                groupslistfinal.add(getMaintenancePeriodType(result.timeperiods))
                                groupslistfinal.add(getMaintenanceSchedule(result.timeperiods))
                                groupslistfinal.add(getMaintenanceDuration(result.timeperiods))
                            }
                        }
                        groupslistfinal.eachWithIndex{ it, i -> println "$i : $it" }
                        return groupslistfinal
                    } else {
                        println("HTTP response error")
                        System.exit(0)
                    }

                    con.disconnect();

                    def getMaintenancePeriodType(period) {
                        type=convertType(period[0].timeperiod_type)

                        return "Period Type - " + type;
                    }

                    def getMaintenanceSchedule(period) {

                        starttime=convertSeconds(period[0].start_time)
                        day=period[0].day

                        return "Schedule - At " + starttime + " on day " + day;
                    }

                    def getMaintenanceDuration(period) {
                        duration=convertPeriod(period[0].period)
                        return "Duration - " + duration + "h";
                    }
                    def convertSeconds(secondsToConvert){
                        long millis = secondsToConvert.toInteger() * 1000;
                        long hours = TimeUnit.MILLISECONDS.toHours(millis);
                        long minutes = TimeUnit.MILLISECONDS.toMinutes(millis) % TimeUnit.HOURS.toMinutes(1);
                        long seconds = TimeUnit.MILLISECONDS.toSeconds(millis) % TimeUnit.MINUTES.toSeconds(1);
                        String format = String.format("%02d:%02d:%02d", Math.abs(hours), Math.abs(minutes), Math.abs(seconds));
                        return format
                    }
                    
                    def convertPeriod(secondsToConvert){
                        long hours = secondsToConvert.toInteger() / 3600;
                        return hours
                    
                    }
                    def convertType(typetoConvert){
                    
                        switch(typetoConvert) {
                            case "0":
                                return "One time only"
                                break;
                            case "2":
                                return "Daily"
                                break;
                            case "3":
                                return "Weekly"
                                break;
                            case "4":
                                return "Monthly"
                                break;
                            default:
                                return " "
                        }
                        }

                    '''
                  ]
                ]
              ],
              [$class: 'ChoiceParameter',
                choiceType: 'PT_SINGLE_SELECT',
                description: 'Machines of each Group',
                filterLength: 1,
                filterable: false,
                name: 'INFO_GROUPS',
                script: [
                  $class: 'GroovyScript',
                  fallbackScript: [
                    classpath: [],
                    sandbox: false,
                    script:
                    'return ["Error"]'
                  ],
                  script: [
                    classpath: [],
                    sandbox: false,
                    script: '''
                    import groovy.json.JsonSlurper
                    import java.net.HttpURLConnection
                    import java.net.URL
                    import java.io.OutputStream

                    //VARIABLES
                    def api_token
                    API_URL = "http://172.18.0.1:8081/api_jsonrpc.php";

                    //JSONS BODY TOKEN
                    GET_TOKEN_PARAMS = "{\\"jsonrpc\\":\\"2.0\\",\\"method\\":\\"user.login\\",\\"params\\":{\\"user\\":\\"Admin\\",\\"password\\":\\"zabbix\\"},\\"id\\":1,\\"auth\\":null}";

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
                    //JSON BODY GROUPS DETAILS

                    GET_GROUPS_PARAMS="{\\"jsonrpc\\":\\"2.0\\",\\"method\\":\\"hostgroup.get\\",\\"params\\":{\\"output\\":\\"extend\\",\\"searchWildcardsEnabled\\":\\"true\\",\\"search\\":{\\"name\\":[\\"JUMIA*\\"]}},\\"auth\\":" + "\\"" + api_token.result + "\\"" + ",\\"id\\":\\"1\\"}"
                    con = new URL(API_URL).openConnection() as HttpURLConnection
                    con.setRequestMethod("GET");
                    con.setRequestProperty("Content-Type", "application/json-rpc");
                    con.setDoOutput(true);
                    os = con.getOutputStream();
                    os.write(GET_GROUPS_PARAMS.getBytes());
                    os.flush();
                    os.close()
                    
                    def groupslistfinal = []
                    if (con.getResponseCode() == 200) {
                        groupslist = new JsonSlurper().parseText(con.getInputStream().getText('UTF-8'))
                        groupslist.result.each { result ->
                            groupslistfinal.add(result.name) }
                        groupslistfinal.eachWithIndex{ it, i -> println "$i : $it" }
                        return groupslistfinal.sort()
                    } else {
                        println("HTTP response error")
                        System.exit(0)
                    }

                    con.disconnect();

                    '''
                  ]
                ]
              ],
              [$class: 'DynamicReferenceParameter', 
                choiceType: 'ET_UNORDERED_LIST', 
                description: 'Details of Group', 
                name: 'INFO_GROUPS_DETAILS', 
                referencedParameters: 'INFO_GROUPS', 
                script: [
                  $class: 'GroovyScript',
                  fallbackScript: [
                    classpath: [],
                    sandbox: false,
                    script:
                    'return ["Error"]'
                  ],
                  script: [
                    classpath: [],
                    sandbox: false,
                    script: '''
                
                    import groovy.json.JsonSlurper

                    def api_token
                    API_URL="http://172.18.0.1:8081/api_jsonrpc.php";
                    GET_TOKEN_PARAMS = "{\\"jsonrpc\\":\\"2.0\\",\\"method\\":\\"user.login\\",\\"params\\":{\\"user\\":\\"Admin\\",\\"password\\":\\"zabbix\\"},\\"id\\":1,\\"auth\\":null}";
                    
                    
                    def con = new URL(API_URL).openConnection() as HttpURLConnection
                    
                    /// GET TOKEN
                    
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
                    con.disconnect()
                    
                    def con2 = new URL(API_URL).openConnection() as HttpURLConnection
                    
                    
                    def group_id
                    GET_GROUPS_PARAMS="{\\"jsonrpc\\":\\"2.0\\",\\"method\\":\\"hostgroup.get\\",\\"params\\":{\\"output\\":\\"extend\\",\\"searchWildcardsEnabled\\":\\"true\\",\\"search\\":{\\"name\\":[\\"JUMIA*\\"]}},\\"auth\\":" + "\\"" + api_token.result + "\\"" + ",\\"id\\":\\"1\\"}"
                    con2.setRequestMethod("GET");
                    con2.setRequestProperty("Content-Type", "application/json-rpc");
                    con2.setDoOutput(true);
                    OutputStream os2 = con2.getOutputStream();
                    os2.write(GET_GROUPS_PARAMS.getBytes());
                    os2.flush();
                    os2.close()
                    
                    
                    def groupslistfinal = []
                    if (con2.getResponseCode() == 200) {
                        groupslist = new JsonSlurper().parseText(con2.getInputStream().getText('UTF-8'))
                        groupslist.result.each { result ->
                            if(result.name.equals(INFO_GROUPS))
                               group_id=result.groupid}
                        con2.disconnect();
                    } else {
                        con2.disconnect();
                        println("HTTP response error")
                        System.exit()
                    }
                    con2.disconnect();
                    
                    def con3 = new URL(API_URL).openConnection() as HttpURLConnection
                    
                    GET_HOSTS_GROUP="{\\"jsonrpc\\":\\"2.0\\",\\"method\\":\\"host.get\\",\\"params\\":{\\"output\\":\\"extend\\",\\"groupids\\":[\\"" + group_id + "\\"]},\\"auth\\":" + "\\"" + api_token.result + "\\"" + ",\\"id\\":\\"1\\"}"
                    con3.setRequestMethod("GET");
                    con3.setRequestProperty("Content-Type", "application/json-rpc");
                    con3.setDoOutput(true);
                    OutputStream os3 = con3.getOutputStream();
                    os3.write(GET_HOSTS_GROUP.getBytes());
                    os3.flush();
                    os3.close()
                    
                    
                    def groupslistfinal2 = []
                    if (con3.getResponseCode() == 200) {
                        groupslist = new JsonSlurper().parseText(con3.getInputStream().getText('UTF-8'))
                        groupslist.result.each { result -> groupslistfinal2.add(result.host)  }
                        groupslistfinal2.eachWithIndex{ it, i -> println "$i : $it" }
                        return groupslistfinal2.sort()
                    
                    } else {
                        println("HTTP response error")
                        System.exit(0)
                    }
                
                    '''
                ]
                ]
              ],
              choice(
                choices: ['CREATE','UPDATE'],
                name: 'OPTION'

              ),
              [$class: 'DynamicReferenceParameter', 
                choiceType: 'ET_FORMATTED_HTML', 
                description: 'Details of Maintenance', 
                name: 'OPTIONS', 
                referencedParameters: 'OPERATION', 
                script: [
                  $class: 'GroovyScript',
                  fallbackScript: [
                    classpath: [],
                    sandbox: false,
                    script:
                    'return ["Error"]'
                  ],
                  script: [
                    classpath: [],
                    sandbox: false,
                    script: '''

                    import groovy.json.JsonSlurper


                    def date = new Date()
                    
                    if ( OPERATION == "CREATE") {
                    
                    MaintenanceHtml = """
                    
                    
                    <ul style="list-style-type:none;padding: 0;margin:0">
                    <label style="font-weight: 500;">Maintenance Name</label>
                    <input type="text" class="setting-input" name="value">
                    <label for="end">Start date:</label>
                    <input type="datetime-local" id="1" class="setting-input" name="value" value=date.format('yyyy-mm-ddTHH:mm', TimeZone.getTimeZone('UTG')) min=date.format('yyyy-mm-ddTHH:mm', TimeZone.getTimeZone('UTG'))/>
                    <label for="end">End date:</label>
                    <input type="datetime-local" id="2" class="setting-input" name="value" value=date.format('yyyy-mm-ddTHH:mm', TimeZone.getTimeZone('UTG')) min=date.format('yyyy-mm-ddTHH:mm', TimeZone.getTimeZone('UTG'))/>
                    <br>
                      </li>
                    </ul>
                    """
                    return MaintenanceHtml
                    }
                    
                    
                    
                    if (OPERATION == "DELETE")
                    {
                    
                    //VARIABLES
                    def api_token
                    API_URL="http://172.18.0.1:8081/api_jsonrpc.php";
                    
                    //JSONS BODY TOKEN
                    GET_TOKEN_PARAMS="{\"jsonrpc\":\"2.0\",\"method\":\"user.login\",\"params\":{\"user\":\"Admin\",\"password\":\"zabbix\"},\"id\":1,\"auth\":null}";
                    
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
                    
                    //JSONS BODY MAINTENANCE LIST
                    GET_MAINTENANCE_LIST="{\"jsonrpc\":\"2.0\",\"method\":\"maintenance.get\",\"params\":{\"output\":\"extend\",\"selectGroups\":\"extend\",\"selectTimeperiods\":\"extend\",\"selectTags\":\"extend\"},\"auth\":" + "\"" + api_token.result + "\"" + ",\"id\":\"1\"}"
                    
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
                            groupslist.result.each { result ->
                        groupslistfinal.add(result.name) }
                        groupslistfinal.eachWithIndex{ it, i -> println "$i : $it" }
                    } else {
                        println("HTTP response error")
                        System.exit(0)
                    }
                    
                    con.disconnect();
                    
                    
                    
                    
                    def html = """
                    
                    <select id="mySelect" class="setting-input" name="value" multiple>
                    """
                    
                    groupslistfinal.each { option ->
                        html += "<option value='${option}'>${option}</option>"
                    }
                    
                    html += """
                    </select>
                    """
                    
                    return html
                    
                    }

                    '''
                  ]
                ]
              ]
            ])
          ])
        }
      }
    }

        stage('Example') {
            steps {
                echo "Selected Country: ${params.OPTION}"
            }
        }


    
  }
  
}
