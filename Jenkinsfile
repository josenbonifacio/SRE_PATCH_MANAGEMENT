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
                    script: '''
                        
                    import groovy.json.JsonSlurper
                    
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
                    
                    //JSONS BODY MAINTENANCE LIST
                    GET_MAINTENANCE_LIST="{\\"jsonrpc\\":\\"2.0\\",\\"method\\":\\"maintenance.get\\",\\"params\\":{\\"output\\":\\"extend\\",\\"selectGroups\\":\\"extend\\",\\"selectTimeperiods\\":\\"extend\\",\\"selectTags\\":\\"extend\\"},\\"auth\\":" + "\\"" + api_token.result + "\\"" + ",\\"id\\":\\"1\\"}"
                    
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
                        return groupslistfinal.sort()
                    } else {
                        println(" HTTP response error")
                        System.exit(0)
                    }
                    
                    con.disconnect();
                    '''
                ]
            ]
        )
        reactiveChoice(
            name: 'INFO_MAINTENANCES_DETAILS',
            description: 'Select maintenance details',
            referencedParameter: 'INFO_MAINTENANCES',
            choiceType: 'BULLETED_ITEMS',
            script: [
                $class: 'GroovyScript',
                fallbackScript: [
                    classpath: [],
                    script: 'return ["Error: Script Not Available"]'
                ],
                script: '''
                    import groovy.json.JsonSlurper
                    import java.net.HttpURLConnection
                    import java.net.URL
                    import java.io.OutputStream
                    import java.util.Date

                    def api_token = INFO_MAINTENANCES
                    API_URL="http://172.18.0.1:8081/api_jsonrpc.php";

                    //JSON BODY MAINTENANCE DETAILS
                    GET_MAINTENANCE_DETAILS="{\\"jsonrpc\\":\\"2.0\\",\\"method\\":\\"maintenance.get\\",\\"params\\":{\\"output\\":\\"extend\\",\\"selectGroups\\":\\"extend\\",\\"selectTimeperiods\\":\\"extend\\",\\"selectTags\\":\\"extend\\"},\\"auth\":" + "\"" + api_token.result + "\"" + ",\\"id\\":\\"1\\"}"
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
                '''
            ]
        )

    }
    stages {


        stage('Example') {
            steps {
                script {
                    echo "Selected maintenance group: ${params.INFO_MAINTENANCES}"
                    echo "Selected maintenance details: ${params.INFO_MAINTENANCES_DETAILS}"
                   /* echo "Selected group: ${params.INFO_Groups}"
                    echo "Selected group details: ${params.GROUP_DETAILS}"
                    echo "Selected operation: ${params.OPERATION}"
                    echo "Selected options: ${params.OPTIONS}"
                    echo "Selected group pick: ${params.GROUP_PICK}"
                    echo "Selected maintenance details: ${params.DETAILS_MAINTENANCE}"*/
                }
            }
        }
    }
}
