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
                    import groovy.json.JsonSlurper
                    import java.net.HttpURLConnection
                    import java.net.URL
                    import java.io.OutputStream
                    import java.util.Date

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
              ]
            ])
          ])

        }
      }
    }
  }
}
