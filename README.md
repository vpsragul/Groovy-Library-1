# Groovy-Library
# Use this repository to store the Groovy samples to be part of the Groovy Library to implement into the EPM solutions
 <header>
	
 # 1.Transfer Data
 
 </header>   
 
##  1.1 Groovy to call the Data Map

Below code can be used to run the data map and can attached in the form 2.

```bash
operation.application.getDataMap("DatamapName").execute(true)
```
- DatamapName : Update the data map name to get executed
- True/blank: Boolean parameter to clear the data before push pushing the data. 
	Selecting  “execute(true)” will clear the data before pushing the data
	Selecting  “execute()” will not clear the data before pushing the data
- Note 1: Consider there is a threshold of 200MB.
- Note 2: This groovy rule will be executed by admin users only.

## 1.2	Groovy to Run Cross pod Data Map

 Below code will convert data map to Smart push and run in groovy
 Crosspod data maps cannot be run as a data map manually and using script. It has to triggered as a smartpush
 
 ``` bash
operation.application.getDataMap("DatamapName").createSmartPush().execute(true)
```

- DatamapName : Update the cross data map name to get executed
- True/blank: Boolean parameter to clear the data before push pushing the data. 
	Selecting  “execute(true)” will clear the data before pushing the data
	Selecting  “execute()” will not clear the data before pushing the data

## 1.3	Groovy to run smart push 

Below code will run smart push attached in the Webforms

```bash
operation.grid.getSmartPush("DatamapName").execute(true)
```
- Data map Name : Update the Smart push data map name to get executed
- True/blank: Boolean parameter to clear the data before push pushing the data. 
	Selecting  “execute(true)” will clear the data before pushing the data
	Selecting  “execute()” will not clear the data before pushing the data
- Note 1: Smart Pushes can error out if the data volume is too big or for other reasons

## 1.4 Groovy to customize members in Data maps and push data

  ```bash
  Map<String, String> overrideMems1 = new HashMap<String, String>();
  overrideMems1.put("Product", "ILvl0Descendants(pd_pd_30000)");
  overrideMems1.put("Entity", "ILvl0Descendants(OR_Global_HQ)");
  overrideMems1.put("ProfitCenter", "ILvl0Descendants(All_Products)");
  if(operation.application.hasDataMap("DatamapName "))
	println "Combination of Datapush: " 
	println  overrideMems1
  operation.application.getDataMap("DatamapName").createSmartPush().execute(overrideMems1,true);

  ```
- Data map Name : Update the Smart push data map name to get executed
- True/blank: Boolean parameter to clear the data before push pushing the data. 
	Selecting  “execute(true)” will clear the data before pushing the data
	Selecting  “execute()” will not clear the data before pushing the data

## 1.5	Groovy to Push data using REST API
```bash
def uvProfitCenter = operation.application.getUserVariable("ProfitCenter").value.name
println("User Profit Center: " + uvProfitCenter)
def uvRevenueCenter = operation.application.getUserVariable("RevenueCenter").value.name
println("User Region: " + uvRevenueCenter)
def uvVersion = operation.application.getUserVariable("OEP_Version").value.name
println("User Version: " + uvVersion)
def uvProduct = operation.application.getUserVariable("Product").value.name
println("User Product: " + uvRevenueCenter)
def EntitySelection = '"ILvl0Descendants(' + uvRevenueCenter + ')"'
def PCSelection = '"ILvl0Descendants(' + uvProfitCenter + ')"'
def VersionList = '"' + uvVersion + '"'
def ProductList = '"' + uvProduct + '"'
if(operation.application.hasDataMap("Financials to RPT - AOP")){
String datamappayload = """
{
"jobType":"PLAN_TYPE_MAP",
"jobName":"Financials to RPT - Actual",
"parameters": {
  	"clearData": true,
  	"overrideMembersMap": {
  	    "Entity": $EntitySelection,
		"Profit Center": $PCSelection
		}
  	}
}
""";
println datamappayload

HttpResponse<String> jsonResponse = operation.application.getConnection("Groovy_Endpoint").post('/jobs')
    .body(datamappayload)
    .asString();
    sleep(3000);
 	String jobID = JsonPath.parse(jsonResponse.body).read('$.jobId')
println "JobID: " + jobID;
HttpResponse<String> pingResponse = operation.application.getConnection("Groovy_Endpoint").get("/jobs/" + jobID).asString()
     int DataMapStatus =  JsonPath.parse(pingResponse.body).read('$.status')
  println "DataMapStatus before while "+DataMapStatus
  // DataMapStatus = -1
  while(DataMapStatus==-1){
  println "DBRefresh inside while:"+DataMapStatus
  pingResponse = operation.application.getConnection("Groovy_Endpoint").get("/jobs/" + jobID).asString()
  DataMapStatus =  JsonPath.parse(pingResponse.body).read('$.status')
  }
  String ErrorMsg = JsonPath.parse(pingResponse.body).read('$.details')
if(DataMapStatus==1)
{
throwVetoException("Data Map Failed with Error: "+ErrorMsg);
}else
{
println "Success";
} }
```
## 1.6	Groovy to export data using Grid Builder:
The code below can be used to export data to a file if the data size is small. If the size is more it will give threshold error.
```bash
The code below can be used to export data to a file if the data size is small. If the size is more it will give threshold error.
Cube cube = operation.application.getCube("OEP_FS")
 cube.createDataExporter().setDateFormat("dd/MM/yyyy")
 .setDelimiter('|')
 .setColumnMemberNames(["USD_SFX"])
 .setRowFilterCriteria(['REVSTAGE_Input'],['OEP_Actual'],['FY24'],['OEP_Working'],['ILvl0Descendants (Total Profit Center)'], ['ILvl0Descendants(OEP_Total Entity)'],['ILvl0Descendants(Total Product)'], ['ILvl0Descendants(YearTotal)'],['ILvl0Descendants(Account)'])
 .setDataExportFormat(CELL_FORMAT)
 .setErrorFileName("Level0DataWithDimHeaderErrors.log")
 .exportDataToFile("Level0DataWithDimHeader.csv")
 ```
## 1.7	Groovy To export data to a file without Threshold Error:
```bash
Application app = operation.application
Cube cube = app.getCube("OEP_FS")
List<Member> PlanningOrder  = app.getDimension("Planning Order").getEvaluatedMembers('ILvl0Descendants("Planning Order")', cube)
List<Member> Scenario       = app.getDimension("Scenario").getEvaluatedMembers("OEP_Forecast", cube)
List<Member> Years          = app.getDimension("Years").getEvaluatedMembers("FY24", cube) 
List<Member> Version        = app.getDimension("Version").getEvaluatedMembers("OEP_Working", cube)
List<Member> ProfitCenter   = app.getDimension("Profit Center").getEvaluatedMembers('ILvl0Descendants("Total Profit Center")', cube)
List<Member> Entity         = app.getDimension("Entity").getEvaluatedMembers('ILvl0Descendants("OEP_Total Entity")', cube)
List<Member> Product        = app.getDimension("Product").getEvaluatedMembers('ILvl0Descendants("Total Product")', cube)
List<Member> Period         = app.getDimension("Period").getEvaluatedMembers('ILvl0Descendants("YearTotal")', cube)
List<Member> Account        = app.getDimension("Account").getEvaluatedMembers('ILvl0Descendants("Account")', cube)
List<Member> Currency       = app.getDimension("Currency").getEvaluatedMembers('ILvl0Descendants("Input Currencies")', cube)

List<Member> rowFilter      = ( PlanningOrder + Scenario + Years + Version + ProfitCenter + Entity + Product + Currency + Account )
/* Planning Order", Scenario, Years, Version, ProfitCenter, Entity, Product, "Currency Type", Period, Account*/  
cube.createDataExporter().setDelimiter(',')
    .setColumnMemberNames(Period)
    .setDataExportFormat(CELL_FORMAT)
    .setRowFilterCriteria([PlanningOrder , Scenario , Years , Version , ProfitCenter , Entity , Product , Currency , Account])
    .setErrorFileName("OEP_FS_ExportErrors.log") 
    .exportDataToFile("Data_Export_Forecast_OEPFS_Acc.txt")println ("Export Data OEP_FS Completed")
```
## 1.8	Groovy to Export data for Entities only to the assigned currency attributes
```bash
Cube cube = operation.application.getCube('OEP_FS') createFilePrintWriter('EPM2MKTModel_Test.txt').withCloseable { writer -> writer.println '"Planning Order'+'"|"'+'Product'+'"|"'+'Profit Center'+'"|"'+'Entity'+'"|"'+'Years'+'"|"'+'Scenario'+'"|"'+'Version'+'"|"'+'Currency'+'"|"'+'Period'+'"|"'+'AC_NET_SALES'+'"|"'+'Revenue_Units_Load"'
Dimension currDim = operation.application.getDimension('Entity', cube) List entities = currDim.getEvaluatedMembers('@RELATIVE("BSC_DIRECT",0)', cube).name /List entities = currDim.getEvaluatedMembers('EN_RV_DOMESTIC', cube).name/
entities.each{ Entity->
Application app = cube.getApplication() 
def entityDim = app.getDimension("Entity", cube) 
def attrFunCurrDim=entityDim.getAttributeDimensions().find{it.name == "ATTR_FUNCTIONAL_CURRENCY"} 
def revcenterLocalCurrency = entityDim.getMember(Entity, cube).getAttributeValue(attrFunCurrDim)
String revCenterAttribute = revcenterLocalCurrency.toString() 
String revCenterFunCurrency = revCenterAttribute.replace("FUN_","")
DataGridDefinitionBuilder builder = cube.dataGridDefinitionBuilder() builder.setSuppressMissingRowsNative(true) 
builder.addPov(['Years', 'Scenario', 'Version'], [['&OEP_CurYr'],['OEP_Forecast'],['OEP_Working']])       
builder.addColumn(['Account'], [ ['"AC_NET_SALES","Revenue_Units_Load"'] ])  
builder.addRow(['Planning Order','Product','Profit Center','Period','Entity','Currency'], [ ['@RELATIVE("Planning Order",0)'], ['@UDA("Product","MKTMODELPRODUCT")'], ['@relative("All_Products",0),@relative("Pharma_Segment",0)'], ['@relative("Yeartotal",0)'], [Entity], [revCenterFunCurrency+',"USD_SFX_Reporting","USD_PFX_Reporting","USD_AFX_Reporting"'] ]) 
DataGridDefinition gridDefinition = builder.build() cube.loadGrid(gridDefinition, false).withCloseable { grid -> grid.dataCellIterator('AC_NET_SALES').each {cell-> if(!cell.missing && cell.data != 0.0){ def dataNetSales = cell.data.round(9) as BigDecimal 
def dataRevUnits = cell.crossDimCell('Revenue_Units_Load').data.round(9) as BigDecimal writer.println '"'+cell.getMemberName('Planning Order')+'"|"'+cell.getMemberName('Product')+'"|"'+cell.getMemberName('Profit Center')+'"|"'+cell.getMemberName('Entity')+'"|"'+cell.getMemberName('Years')+'"|"'+cell.getMemberName('Scenario')+'"|"'+cell.getMemberName('Version')+'"|"'+cell.getMemberName('Currency')+'"|"'+cell.getMemberName('Period')+'"|'+dataNetSales+'|'+dataRevUnits } } } }
}
```
## 1.9	Groovy to Push Data from ASO to ASO using RTP parameters:
```bash
/*RTPS: {RTP_Year} {RTP_Scenario} {RTP_Source_Version} {RTP_Target_Version}*/
def date = new Date()
def dateFormat = "yyyyMMdd'_'HHmmss"
def curDate = date.format(dateFormat, TimeZone.getTimeZone('UTC'))
String Year = rtps.RTP_Year.toString()
String Scenario = rtps.RTP_Scenario.toString()
String Source_Version = rtps.RTP_Source_Version.member.name
String Target_Version = rtps.RTP_Target_Version.member.name
String fileName = curDate + "_TEST"
 Cube cube = operation.application.getCube("FIN_RPT")
 List<String> ListOfMembers = operation.application.getDimension("DimensionName",cube).getEvaluatedMembers("ILvl0Descendants(ParentMember),Member" ,cube)*.name
println "Executed by ${operation.user.fullName}"
/*===1. Export data from FIN_RPT to ZIP FILE cube===*/
 cube.createDataExporter().setDateFormat("dd/MM/yyyy")
 .setDelimiter('	')
 .setColumnMemberNames([ListOfMembers])
 .setRowFilterCriteria("${Year}","${Scenario}","${Source_Version}")
 .setDataExportFormat(CELL_FORMAT)
 .setErrorFileName("${fileName}.log")
 .exportDataToFile("${fileName}.txt") 
/*====2. TRANSFORM VERSION ===*/
String[] targetRow = []
String targetVersion = ''
 csvIterator("${fileName}.txt",'	').withCloseable() { reader ->
   reader.each { String[] values ->
      // println(Arrays.toString(values))
       for(int i=0;i<values.size();i++){  
       		if(i==3){
                targetVersion +=  Target_Version + "	"				 
					}
             else{
             	targetVersion +=  values[i] + "	"
             }             				         
       }       
       targetRow += targetVersion
       //targetRowFinal = targetRow.collect{ '"' + it + '"'}
       targetVersion = ''
   }
 }
println "targetRow: " + targetRow
String[] targetRowFinal
String targetFileName
targetFileName = "Transformed_"+"${fileName}"
csvWriter("${targetFileName}.txt",'\t',true).withCloseable() { out ->	
       for(int i=0;i<targetRow.size();i++){  
			targetRowFinal = targetRow[i].split('\t',-2)
   			out.writeNext targetRowFinal[0],targetRowFinal[1],targetRowFinal[2],targetRowFinal[3],targetRowFinal[4],targetRowFinal[5],targetRowFinal[6],        				targetRowFinal[7],targetRowFinal[8],targetRowFinal[9],targetRowFinal[10],targetRowFinal[11],targetRowFinal[12],targetRowFinal[13],                            targetRowFinal[14],targetRowFinal[15],targetRowFinal[16],targetRowFinal[17],targetRowFinal[18]
		}	
 }
/*===3. Import data from ZIP FILE to REV_RP_A cube===*/
Cube cube1 = operation.application.getCube("REV_RP_A")
cube1.createDataImportRequest().importDataFromFile("${targetFileName}.txt", CELL_FORMAT, '	')
println "Data imported successfully to REV_RP_A cube!"
println "The ${targetFileName}.txt is generated in the Inbox/Outbox Explorer"
```
# 2	Virtual Grid
# 3	Execute Job
## 3.1	Groovy to Run Pipeline using Rest API
```bash
HttpResponse<String> jsonResponse =operation.application.getConnection("ConnectionName").post()
.body(json(["jobName":"PipelineName", "jobType":"pipeline",
  "variables":
  ["SEND_MAIL":"No"]              
  ]))
.asString();
println(jsonResponse.body)
println(jsonResponse.status)
println(jsonResponse.statusText)
```
## 3.2	Groovy to Run Data Management Data Load Rule using Rest API
Create a connection using “Other Web Services Provider” called “DM”.
 
 ```bash
/*RTPS: */
Connection conDM = operation.application.getConnection("DM")
HttpResponse<String> jsonResponseDM1 = conDM.post()
.header("Content-Type", "application/json")
.body("""
{"jobType":"DATARULE", "jobName":"NAME OF THE DATA LOAD RULE", "startPeriod":"${curStartPeriod}", "endPeriod":"${curEndActPeriod}",
           "importMode":"REPLACE", "exportMode":"STORE_DATA"}
 """).asString();
boolean pushDataToPlanning1 = awaitCompletion1(jsonResponseDM1, "DM", "Push Top level data to No Members")
// Wait for DM job to be completed
def awaitCompletion1(HttpResponse<String> jsonResponseDM1, String connectionName, String operation) {
    final int IN_PROGRESS = -1
    if (!(200..299).contains(jsonResponseDM1.status))
    throwVetoException("Error occured: $jsonResponseDM1.statusText")
    // Parse the JSON response to get the status of the operation. Keep polling the DM server until the operation completes.
    ReadContext ctx = JsonPath.parse(jsonResponseDM1.body)
    int status = ctx.read('$.status')
    for(long delay = 50; status == IN_PROGRESS; delay = Math.min(1000, delay * 2)) {
        sleep(delay)
        status = getJobStatus1(connectionName, (String)ctx.read('$.jobId'))
    }
println("$operation ${status == 0 ? "successful" : "failed"}.\n")
    return status == 0
}
// Poll the DM server to get the job status
int getJobStatus1(String connectionName, String jobId) {
    HttpResponse<String> pingResponse = operation.application.getConnection(connectionName).get("/" + jobId).asString()
    return JsonPath.parse(pingResponse.body).read('$.status')
}
```
## 3.3	Groovy to Run a Business Rule using Rest API
Create a connection using “Other Web Services Provider” called “Financials_Connection”.
```bash 
Connection conFin = operation.application.getConnection("Financials_Connection")
HttpResponse<String> jsonResponse = conFin.get("/rest/v3/applications/<APPLICATION_NAME>/substitutionvariables").asString()
Map <String,Collection<?>> response = new JsonSlurper().parseText(jsonResponse.body) as Map
def prevDate = new Date() - 14
def cPrevDay = "${prevDate.format('MMM-dd')}"
def cPrevDayYear = "FY" + "${prevDate.format('yy')}"
println "Start Day: " + cPrevDay
println "Start Year: " + cPrevDayYear
try { 
Map importDataToTargetMap = new HashMap()
importDataToTargetMap['aggregateEssbaseData'] = false
importDataToTargetMap['cellNotesOption'] = 'Overwrite'
importDataToTargetMap['dateFormat'] = "DD/MM/YYYY"
importDataToTargetMap['dataGrid'] = response
jsonResponse = conFin.post("/rest/v3/applications/<APPLICATION_NAME>/jobs")
 .header("Content-Type", "application/json")
 .body("""
{
    "jobType":"RULES",
    "jobName":"<BUSINESS_RULE_NAME>",
    "parameters":
    {
        "rtp_Day":"${cPrevDay}",
        "rtp_Year":"${cPrevDayYear}"
    }
}
 """).asString()
// Code to wait until all the rules end for rollup workforce
def json = new JSONObject(jsonResponse.body)
String jobId = json.getInt("jobId").toString()
json = new JSONObject(jsonResponse.body)
int currentStatus = json.getInt("status")
while(currentStatus == -1){
    jsonResponse = conFin.get("/rest/v3/applications/<APPLICATION_NAME>/jobs/${jobId}")
    .header("Content-Type", "application/json")
    .asString()
    json = new JSONObject(jsonResponse.body)
currentStatus = json.getInt("status")
sleep(1000) // 1 second
}
// check to see if there is a status
String msg = ""
if(response["status"] != null){
    msg = "An error was returned with a status of ${response['status']}.  ${response['localizedMessage']}.  Check the job console for further information."
println msg
    throwVetoException(msg)
}
else if(response["numRejectedCells"] != 0){
    msg = "The process completed successfully"
println msg
}
else{
}
println "Process ended at ${(new Date()).format("yyyy-MM-dd','HH:mm:ss:SSS z", TimeZone.getTimeZone('PST'))}"
} catch(Exception e) {
    throwVetoException("Exception: ${e}")  
}
```
## 3.1	Groovy to generate user groups report and make security changes.
```bash
HttpResponse<String> jsonUserGroupReportExport= operation.application.getConnection("internalConnection").post('/interop/rest/security/v1/usergroupreport/?filename=User_Group_Report.csv')
    .header("Content-Type","application/x-www-form-urlencoded")
    .asString()
List<String> users = [] // Variable definition to store users in this list
// Retrieve users from the User and Group report file
csvIterator('User_Group_Report.csv').withCloseable() { reader ->
    reader.each { values ->
        if ((values[5] == "GR_Capex_Escritura")) { // Check if the user belongs to the 'GR_Capex_Escritura' group
            users.add(values[0]) // Add the user's login to the list
        }    }  }
// Create a UTF-8 formatted file to execute commands for adding and removing users
createFilePrintWriter('write_to_read_capex.csv', 'UTF-8', false).withCloseable() { out ->
       out.println 'User Login' // Write the header row
       // Write each user login into the file
       for(int i in 0..users.size()-1 ){    
           out.println users[i].toString() 
       }
}
// Command to add users to the read-only group
HttpResponse<String> jsonAddUsersToGroupResponse = operation.application.getConnection("internalConnection")
.put("/interop/rest/security/v1/groups")
.header("Content-Type", "application/x-www-form-urlencoded")
.body('jobtype=ADD_USERS_TO_GROUP' + '&' + 'groupname=GR_Capex_Lectura' + '&' + 'filename=write_to_read_capex.csv')
.asString()
println jsonAddUsersToGroupResponse.body // Print the response from the API call

// Command to remove users from the write-access group
HttpResponse<String> jsonRemoveUsersToGroupResponse = operation.application.getConnection("internalConnection")
.put("/interop/rest/security/v1/groups")
.header("Content-Type", "application/x-www-form-urlencoded")
.body('jobtype=REMOVE_USERS_FROM_GROUP' + '&' + 'filename=write_to_read_capex.csv' + '&' + 'groupname=GR_Capex_Escritura')
.asString()
println jsonRemoveUsersToGroupResponse.body // Print the response from the API call
```
# 4	Iteration
## 4.1	Groovy to Clear Zeros in Application:
```bash
	The below script is combined with Calcscript to clear out zeros in the Application by iterating through account members. 
Cube cube = operation.application.getCube("CubeName")
Dimension AccountDim = operation.application.getDimension("Account", cube)
// Store the list of companies into a collection
def Account = AccountDim.getEvaluatedMembers("ILvl0Descendants(Account)", cube) as String[]
// Execute a Data Map on each company/currency
def[i]=0
String calcScript;
calcScript = '''
SET UPDATECALC OFF;
SET AGGMISSG ON;
SET FRMLBOTTOMUP ON;
SET FRMLRTDYNAMIC OFF;
/*SET CALCTASKDIMS 4;
SET CALCPARALlEL 20;*/
FIXPARALLEL(7,@relative(ProfitCenter,0))
     /* FIX (@relative(Account,0))*/
      		  "'''+AccountDim[i]+'''" (
              @CALCMODE(BLOCK);
              IF ( (@CURRMBR(Account)==0) or (@CURRMBR(Account)==-0 ))
                    @CURRMBR(Account)=#Missing;	
              ENDIF
                  )
              CLEARBLOCK EMPTY;   
      /*ENDFIX*/                
ENDFIXPARALLEL
'''
println calcScript;
return calcScript;    
```
## 4.2	Concatenate members with text fields to be imported as a description in a new text field
```bash
/* RTPS: {varYear}, {varScenario}, {varCube} */
// 1. Initialize lists to store extracted data
List<String> productList = []
List<String> entityList = []
List<String> periodList = []
List<String> conceptList = []
// 2. Iterate through the data grid and collect values
operation.grid.dataCellIterator('SM_Concepts').each { cell ->
    if (!cell.missing) {
        productList << cell.getMemberName("Product")  // Get product member name
        entityList << cell.entityName                // Get entity member name
        periodList << cell.periodName                // Get period member name
        conceptList << cell.formattedValue           // Get text field value from SM_Conceptos
    }
}
// 3. Concatenate lists to build unique keys
List<String> keyList = []
// 4. Ensure all lists have the same size before concatenation
if (productList.size() == entityList.size() && periodList.size() == conceptList.size()) {
    for (int i = 0; i < productList.size(); i++) {
        String concatenatedKey = "${productList[i]}-${entityList[i]}-${periodList[i]}-${conceptList[i]}"
        keyList.add(concatenatedKey)
    }
}
// 5. Create a CSV file stored in the inbox to be used for data import
csvWriter('Import-Keys.csv').withCloseable { writer ->
    // Header for the import file (based on data export format)
    writer.writeNext("Product", "What If", "Point-of-View", "Data Load Cube Name")
    // Iterate through all collected data and populate the CSV
    for (int i in 0..<productList.size()) {
        writer.writeNext(
            productList[i].toString(), 
            keyList[i].toString(), 
            "${entityList[i]},${periodList[i]},BaseData,${varScenario},${varYear},Risk",
            varCube
        )
        writer.writeNext(
            productList[i].toString(), 
            "1", 
            "${entityList[i]},${periodList[i]},BaseData,${varScenario},${varYear},${conceptList[i]}",
            varCube
        )
    }
}
// 6. Execute a data import job using REST API

HttpResponse<String> jsonResponse = operation.application.getConnection("internalConnection")
    .post("/HyperionPlanning/rest/v3/applications/" + operation.application.name + "/jobs")
    .header("Content-Type", "application/json")
    .body(json(["jobType": "IMPORT_DATA", "jobName": "Import Keys"]))
    .asString()
    ```
# 5	Send Email
## 5.1	Groovy to send a Report to an Email Notification
- Create a connection using “Other Web Services Provider” called “Financials_Connection”.
  
- Create a Report.
 
- Create a Bursting using the prior Report.
 
- Create a Job Schedule. 

- Create the groovy rule.
```bash
/*RTPS: */
Connection conFin = operation.application.getConnection("Financials_Connection") 
HttpResponse<String> jsonResponse = conFin.get("/rest/v3/applications/<APPLICATION_NAME>/substitutionvariables").asString()
Map <String,Collection<?>> response = new JsonSlurper().parseText(jsonResponse.body) as Map
try { // Call Post Load rule on Demand to calculate Opening Balance from Saturday to Monday
    //Build import json
Map importDataToTargetMap = new HashMap()
importDataToTargetMap['aggregateEssbaseData'] = false
importDataToTargetMap['cellNotesOption'] = 'Overwrite'
importDataToTargetMap['dateFormat'] = "DD/MM/YYYY"
importDataToTargetMap['dataGrid'] = response

jsonResponse = conFin.post("/rest/v3/applications/<APPLICATION_NAME>/jobs")
 .header("Content-Type", "application/json")
 .body("""
{
    "jobType":"Execute Bursting Definition",
    "jobName":"TRE Approved Send to ERP Bursting Job",
    "parameters":
    {
        "burstingDefinitionName":"/Library/<FOLDER1>/<SUBFOLDER11>/<REPORT_NAME>"
    }
}
 """).asString()

// Code to wait until all the rules end for rollup workforce
def json = new JSONObject(jsonResponse.body)

String jobId = json.getInt("jobId").toString()
json = new JSONObject(jsonResponse.body)
int currentStatus = json.getInt("status")
while(currentStatus == -1){
    jsonResponse = conFin.get("/rest/v3/applications/<APPLICATION_NAME>/jobs/${jobId}")
 	   	.header("Content-Type", "application/json")
    	.asString()
    json = new JSONObject(jsonResponse.body)
	currentStatus = json.getInt("status")
	sleep(1000) // 1 second
}
// check to see if there is a status
String msg = ""
if(response["status"] != null){
    msg = "An error was returned with a status of ${response['status']}.  ${response['localizedMessage']}.  Check the job console for further information."
	println msg
    throwVetoException(msg)
}
else if(response["numRejectedCells"] != 0){
	//println "${response['numAcceptedCells']} cells were loaded."
    msg = "The process completed successfully"
	println msg
    //throwVetoException(msg)    
}
else{
	//println "${response['numAcceptedCells']} cells were loaded."
}	
println "Process ended at ${(new Date()).format("yyyy-MM-dd','HH:mm:ss:SSS z", TimeZone.getTimeZone('PST'))}"
} catch(Exception e) {
    //println("Exception: ${e}")
    throwVetoException("Exception: ${e}")  
}
```
## 5.2	Take data from a virtual grid transform it to a HTML table and attached it to a email.
```bash
def name = [] // Variable to store the full name of the user executing the script
user = operation.user.getName() // Get the login username(email)
name = operation.user.getFullName() // Get the full name of the logged-in user
Cube cube = operation.application.getCube("EGR") // Get the cube named "EGR"
DataGridDefinitionBuilder builder = cube.dataGridDefinitionBuilder()
// Suppress missing rows and columns in the grid
builder.setSuppressMissingRowsNative(true)
builder.setSuppressMissingColumns(true)
// Define the POV (Point of View) for data retrieval
builder.addPov(['Cecos', 'Custom1', 'Segment', 'Partners','Scenario', 'Version', 'Years', 'Account','Period','Components','Cebes'], 
			  [['Total Cecos'],['Capex Categories'],['Capex Projects'],['Total Partners Capex'],['AOP'],['Working'],['&Anio_Ppto'],['Capex Concepts'],['YearTotal'],['Project Type'],['Total Areas']])
// Define columns
builder.addColumn(['Metrics'], [['USD']])
// Define rows (Entities)
builder.addRow(['Entity'], [['Ilvl0Descendants("G_TOTAL")']])
DataGridDefinition gridDefinition = builder.build() // Build the grid definition
List<Double> monto = [] // List to store financial amounts
List<String> codigo = [] // List to store entity codes
// Load data from the cube
cube.loadGrid(gridDefinition, false).withCloseable { grid -> 
    grid.dataCellIterator().each { 
        if (!it.missing) { // Check if the cell contains data
            monto << it.data // Store the value in 'monto' list
            codigo << it.getMemberName("Entity") // Store the entity code
        }
    }
}

// Retrieve entity aliases for better readability
List<String> entity = codigo.collect { cod ->
    Dimension dim = operation.application.getDimension("Entity", cube)
    Member mbr = dim.getMember(cod, cube)
    if (mbr) {
        mbr.getAlias("Default") // Get the entity alias (default name)
    } else {
        ""
    }  }
// Format the amounts into currency format
List<String> montoFormateado = monto.collect { montoItem ->
    def formatoMoneda = "\$ " + String.format('%,.2f', montoItem)
    formatoMoneda
}
String sendTo = user + "@SAMPLE.com" // Construct the recipient's email
String subject = "SAMPLE SUBJET" // Email subject
// Start constructing the email body
String generateEmailBody = "<!DOCTYPE html><html><body>" +
        "<h1 style='font-weight:bold; font-size: 18px;'>Compass Summary Capex</h1>" +
        "<p>Dear $name,</p>" +
        "<p>Below is a summary of the current total CapEx budget by entity in US dollars. For more details, please check the attached file.</p>" +
                "<!-- HTML Table -->" +
        "<table border='1' cellpadding='10' cellspacing='0' style='border-collapse:collapse;'>" +
            "<tr>" +
                "<th style='font-weight:bold;'>Code</th>" +
                "<th style='font-weight:bold;'>Entity</th>" +
                "<th style='font-weight:bold;'>Amount</th>" +
            "</tr>" +
            // Dynamically add rows to the table
            (0..<codigo.size()).collect { index ->
                "<tr>" +
                    "<td>${codigo[index]}</td>" +
                    "<td>${entity[index]}</td>" +
                    "<td>${montoFormateado[index]}</td>" +
                "</tr>"
            }.join("") +
        "</table>" +
        "<p>For any additional details, please contact your administrator.</p>" +
        "<p style='font-style:italic;'>Best regards,</p>" +        
        
        "<!-- Additional Image -->" +
        "<div style='float: left; margin-right: 5px;'>" +
          "<img src='URL FOR HTML WITH IMAGE' />" +
        "</div>" +
        
        "<p style='font-weight:bold; color: white;'>________</p>" +
        "<div style='font-style:italic; text-align:center; color: grey; font-size: 12px;'>" +
            "This email was sent automatically. Please do not reply." +
        "</div>" +
        
    "</body></html>"

String attachments = "Capex-AOP Total.zip" // Attachments for the email

// Send the email using the internal connection
HttpResponse<String> jsonResponse = operation.application.getConnection("internalConnection").post("/interop/rest/v2/mails/send")
    					       .header("Content-type", "application/json")
                                	       .body(json("to": sendTo, "subject": subject, "body": generateEmailBody, "parameters": ["attachments": attachments]))
                                           .asString()
```
# 6	Data Validation
## 6.1	Color data validation and pop-up warnings
```bash
/* RTPS: {varUser} */
// 1. Define lists to store extracted members and data values
List<String> productMembers = [] // Store product member names
List<Double> cellData = []
// 2. Retrieve the full name of the executing user
String currentUser = operation.user.fullName 
// 3. Define an error message template
String errorMessage = "Dear $currentUser,\nThe entered value is NOT allowed. Please correct the following data points:"
// 4. Iterate through grid cells for specific dimensions ("What If" and "Mar")
operation.grid.dataCellIterator('What If', 'Mar').each { cell ->
    // 5. Apply background color based on conditions
    // If the cell is missing data, set background color to green
    if (cell.missing) {  
        cell.bgColor = 0x92D050 // Green
    }
    // If the cell's data is less than or equal to the value in the 'Jan' cross-dimension cell
    else if (cell.data <= cell.crossDimCell('Jan').data && !cell.missing && !cell.crossDimCell('Jan').missing) {
        cell.bgColor = 0xDAE6F2 // Blue
    }
    // If the cell value is between 10 and 15, set background color to yellow
    else if (cell.data >= 10 && cell.data <= 15) {
        cell.bgColor = 0xFFFF00 // Yellow
    }
    // If the value is outside the allowed range (greater than 15 or less than 0), throw an error
    else if (cell.data > 15 || cell.data < 0) {
        throwVetoException(errorMessage + productMembers)
    }
}
```
# 7	Metadata Changes
## 7.1	Groovy to Refresh DB using REST API
	The below code runs the DB refresh job created in the application by using REST API and connection.
```bash
String strParams = '{"jobType":"CUBE_REFRESH","jobName":"RefreshDatabase"}'
HttpResponse<String> jsonResponse = operation.application.getConnection("Jobs").post().body(strParams).header("Content-Type", "application/Json").asString()
	“Jobs” is the REST API connection name
	“RefreshDatabase” is the job created in Overview to refresh DB
 ```
## 7.2	Groovy to bring metadata for a specific dimension from EDM
```bash
/* RTPS: {varEDMApplication}, {varEDMDimension}, {varEDMConnection}, {varPLNImportJob} */
// 1. Initialize input variables
String EDMapplicationName = varEDMApplication  // Registered Planning application name
String EDMdimensionName   = varEDMDimension    // Managed dimension name
String EDMconnectionName  = varEDMConnection   // Connection name for Planning application
String EDMexportFileName  = EDMdimensionName + ".csv"  // Dynamically set export file name
// Planning instance (generalized)
String PLNconnectionNameForEDM = "EDM_Connection"      // Named connection for EDM instance
String PLNconnectionNameForPLN = "internalConnection"  // Named connection for this Planning instance
String PLNimportDimJobName = varPLNImportJob           // Job name for importing the managed dimension into Planning
// 2. Retrieve the application and dimension IDs for the Planning application and managed dimension in EDM
HttpResponse<String> jsonResponse = operation.application.getConnection(PLNconnectionNameForEDM)
    .get("/epm/rest/v1/applications").asString()
String appJSON = JsonPath.parse(jsonResponse.body).read('$.items[?(@.name == "' + EDMapplicationName + '")]')
String[] EDMapplicationID = JsonPath.parse(appJSON).read('$.[?(@.name == "' + EDMapplicationName + '")].id')
String[] EDMdimensionID = JsonPath.parse(appJSON).read('$..dimensions[?(@.name == "' + EDMdimensionName + '")].id')
// 3. Retrieve the connection ID for the registered Planning application in Enterprise Data Management
jsonResponse = operation.application.getConnection(PLNconnectionNameForEDM)
    .get("/epm/rest/v1/applications/" + EDMapplicationID[0] + "/connections").asString()
String[] EDMconnectionID = JsonPath.parse(jsonResponse.body).read('$.items[?(@.name == "' + EDMconnectionName + '")].id')
// 4. Export the managed dimension from EDM to the Planning instance inbox
jsonResponse = operation.application.getConnection(PLNconnectionNameForEDM)
    .post("/epm/rest/v1/dimensions/" + EDMdimensionID[0] + "/export/connection")
    .header("Content-Type", "application/json")
    .body(json(["connectionId": EDMconnectionID[0], "fileName": EDMexportFileName]))
    .asString()
String[] jobURL = JsonPath.parse(jsonResponse.body).read('$.links[?(@.rel == "results")].href')
String EDMexportJobID = jobURL[0].substring(jobURL[0].lastIndexOf("/") + 1)
// 5. Check the EDM Dimension Export job status
while (getEDMexportJobStatus(PLNconnectionNameForEDM, EDMexportJobID) != "COMPLETED") {
    sleep(1000)
}
// Get the EDM Dimension Export Job status
String getEDMexportJobStatus(String connectionName, String jobId) {
    HttpResponse<String> pingResponse = operation.application.getConnection(connectionName)
        .get("/epm/rest/v1/jobRuns/" + jobId + "/result").asString()
        if (pingResponse.status != 200)
        throwVetoException("Error occurred: $pingResponse.statusText")
        return JsonPath.parse(pingResponse.body).read('$.status')
}
// 6. Run a job in Planning to import the dimension metadata from the inbox.


jsonResponse = operation.application.getConnection(PLNconnectionNameForPLN)
    .post("/HyperionPlanning/rest/v3/applications/" + EDMapplicationName + "/jobs")
    .header("Content-Type", "application/json")
    .body(json(["jobType": "IMPORT_METADATA", "jobName": PLNimportDimJobName, "parameters": ["refreshCube": "false"]]))
    .asString()

// Wait for DM job to be completed
def awaitCompletion(HttpResponse<String> jsonResponse, String connectionName, String operation) {
    final int IN_PROGRESS = -1
    if (!(200..299).contains(jsonResponse.status))
        throwVetoException("Error occurred: $jsonResponse.statusText")
    
    // Parse the JSON response to get the status of the operation. Keep polling the DM server until the operation completes.
    ReadContext ctx = JsonPath.parse(jsonResponse.body)
    int status = ctx.read('$.status') 
    for (long delay = 50; status == IN_PROGRESS; delay = Math.min(1000, delay * 2)) {
        sleep(delay)
        status = getJobStatus(connectionName, ctx.read('$.jobId'))
    }    
    println("$operation ${status == 0 ? "successful" : "failed"}.\n")
    return status == 0
}
// Poll the DM server to get the job status
int getJobStatus(String connectionName, String jobId) {
    HttpResponse<String> pingResponse = operation.application.getConnection(connectionName)
        .get("/HyperionPlanning/rest/v3/applications/" + EDMapplicationName + "/jobs/" + jobId)
        .asString()
    
    return JsonPath.parse(pingResponse.body).read('$.status')
}
```
# 8	Form Validation
## 8.1	Groovy To Block the cells in Forms

 The below code should be attached to the form to block the cells. 
```bash
operation.grid.dataCellIterator('OFS_Adjustment (+/-)').each { cell ->
	if (cell.getMemberName("Account") != 'Revenue_Units_Load' && cell.getMemberName("Account") != 'AC_ASP' && cell.getMemberName("Account") != 'AC_NONREV_COGS_UNITS'){
    cell.setForceReadOnly(true);
}}	
```
## 8.2	Groovy to Pick Member from the form and run calc only for members on the Form
```bash
def uvRevenueCenter = operation.application.getUserVariable("RevenueCenter").value.name
println("User Revenue Center: " + uvRevenueCenter)
def uvVersion = operation.application.getUserVariable("OEP_Version").value.name
println("User Version: " + uvVersion)
List<String> manyprofitcenters = []; // List to store base products
String calcScript;
operation.grid.dataCellIterator().each {cell ->   
	manyprofitcenters << cell.getMemberName("Profit Center")
}
List<String> profitcenter = manyprofitcenters.unique() // Get unique sorted products
calcScript = '''
SET UPDATECALC OFF;
FIX (/*Scenario*/	"OEP_Plan"
/*Account*/ 		"Revenue_Units_Load"
/*Currency*/		"USD",
/*Entity*/		 	"'''+uvRevenueCenter+'''",
/*Years*/			&OEP_PlanStartYr,
/*Version*/			
/*Planning Order*/	"OFS_Direct Input",
/*Period*/			@IDescendants("YearTotal"),
/*Product*/			@IDescendants("pd_total_product")
)
'''
if (profitcenter.size() > 0){
    profitcenter.each { profitc ->
	calcScript = calcScript + '''
	FIX(/*Profit Center*/"'''+profitc+'''")
	"'''+uvVersion+'''"(
	"'''+uvVersion+'''" = "AC_NET_SALES" / "AC_ASP";
	)
	ENDFIX  
	'''
	}    
//Join calcScripts
calcScript = calcScript + '''
ENDFIX
'''
println calcScript;
return calcScript;	
}
```
## 8.3	Groovy to validate the members in the form
```bash
def user = operation.user.getFullName()
Cube cube = operation.application.getCube("OEP_FS") // Get the cube for OEP_FS
Dimension dim = operation.application.getDimension("Product", cube) // Get the Product dimension
//Function to generate Member Alias from member name to show in messages
String getMemberAlias(String member) {
Cube cube = operation.application.getCube("OEP_FS")
Dimension dim = operation.application.getDimension("Product", cube)
Member mbr = dim.getMember(member, cube)
Map mbrProperties = mbr.toMap()

if(mbrProperties["Alias: Default"] == null){
return (mbr.getName())
}

else {
    return (mbr.getAlias("Default"))    
} }

// Validation to advise at user that is not possible to put an adjustment when the product is tagged as no change
operation.grid.dataCellIterator('BegBalance').each { cell ->
if (cell.formattedValue == 'No Change')
  { 
	operation.grid.dataCellIterator().each { item ->
	if (item.edited && !item.missing && (item.getMemberName("Account") == "AC_NET_SALES" ||  item.getMemberName("Account") == "Revenue_Units_Load"))
	{
    List<String> getmembers = dim.getEvaluatedMembers("Ancestors(${item.getMemberName("Product")})", cube)*.name
    getmembers.each{productevaluated ->
    if(productevaluated == cell.getMemberName("Product"))
    {throwVetoException("Dear ${user}, you cannot enter an adjustment value for a product that has an ancestor tagged as 'No Change'. Please review the following product: " + getMemberAlias(item.getMemberName("Product")) + " his ancestor is: " + getMemberAlias(cell.getMemberName("Product")) );}
	else if ( item.getMemberName("Product") == cell.getMemberName("Product") )
    {throwVetoException("Dear ${user}, you cannot enter an adjustment value for a product tagged as 'No Change'. Please review the following product: " + getMemberAlias(cell.getMemberName("Product")) );}
		} 
	}
       
	else if (item.edited && !item.missing && (item.getMemberName("Account") == "AD_ASP"))
	{
    List<String> getmembers = dim.getEvaluatedMembers("Ancestors(${item.getMemberName("Product")})", cube)*.name
    getmembers.each{productevaluated ->
    if(productevaluated == cell.getMemberName("Product"))
    {throwVetoException("Dear ${user}, you cannot enter an adjustment percentage for a product that has an ancestor tagged as 'No Change'. Please review the following product: " + getMemberAlias(item.getMemberName("Product")) + " his ancestor is: " + getMemberAlias(cell.getMemberName("Product")) );}
	else if ( item.getMemberName("Product") == cell.getMemberName("Product") )
    {throwVetoException("Dear ${user}, you cannot enter an adjustment percentage for a product tagged as 'No Change'. Please review the following product: " + getMemberAlias(cell.getMemberName("Product")) );}
		}    
	}
	}
  }	
}
```
## 8.4	Groovy to Run Calc only for Edited Cells
```bash
def uvProfitCenter = operation.grid.pov.find {it.dimName =='Profit Center'}.essbaseMbrName
println("User Profit Center: " + uvProfitCenter)
def uvRevenueCenter =  operation.application.getUserVariable("RevenueCenter").value.name
println("User Revenue Center: " + uvRevenueCenter)
def uvVersion = operation.application.getUserVariable("OEP_Version").value.name
println("User Version: " + uvVersion)
def calcScript = new StringBuilder()
calcScript<<"""
SET AGGMISSG ON;
VAR allocatedValues;
VAR remainingAllocation;
VAR allocatedUnits;
FIX(&OEP_FCSTStartYr, "OEP_Forecast", "$uvRevenueCenter", "USD", "OFS_Direct Input", "$uvVersion", @RELATIVE("$uvProfitCenter", 0))
"""
// Iterate on the edited cells and process the values entered
operation.grid.dataCellIterator({DataCell cell -> cell.edited}, MemberNameType.ESSBASE_NAME).each {
	DataCell cell ->
      String cellProduct = '"' + cell.getMemberName("Product") + '"'
      String cellPeriod = '"' + cell.getMemberName("Period") + '"'
      println(cellProduct + " " + cellPeriod)
      if(cellPeriod.contains("Jan") || cellPeriod.contains("Feb") || cellPeriod.contains("Mar") || cellPeriod.contains("Apr") || cellPeriod.contains("May") || cellPeriod.contains("Jun") || cellPeriod.contains("Jul") || cellPeriod.contains("Aug") || cellPeriod.contains("Sep") || cellPeriod.contains("Oct") || cellPeriod.contains("Nov") || cellPeriod.contains("Dec")){
		calcScript<<"""
			FIX($cellPeriod)
				"ACS_PRICE_VAR_ADJ"(
					allocatedValues = #MISSING;
					remainingAllocation = #MISSING;
                    allocatedUnits = #MISSING;
				);
			FIX(@RELATIVE($cellProduct, 0))
				"ACS_PRICE_VAR_ADJ"(
					IF("No Account"==1)
						allocatedValues = allocatedValues + "ACS_PRICE_VAR_ADJ";
                        allocatedUnits = allocatedUnits + "ACS_ASP_IMPACT";
					ENDIF
				);
			ENDFIX
			"ACS_PRICE_VAR_ADJ"(
				remainingAllocation = $cellProduct - allocatedValues;
			);
			FIX(@RELATIVE($cellProduct, 0))
				"ACS_PRICE_VAR_ADJ"(
					IF("No Account"<>1)
						"ACS_PRICE_VAR_ADJ" = remainingAllocation * ("ACS_ASP_IMPACT" / ("ACS_ASP_IMPACT"->$cellProduct - allocatedUnits ));
						"No Account" = 1;
					);
				ENDFIX
			ENDFIX
		"""
    	}
	}
calcScript<<"""
	FIX(@RELATIVE("pd_product", 0))
       	CLEARDATA "No Account";
	ENDFIX
    AGG("Product");
    FIX(@REMOVE(@IDESCENDANTS("pd_all_products"), @RELATIVE("pd_all_products", 0)))
    	"AC_ASP" = "AC_NET_SALES" / "Revenue_Units_Load";
    ENDFIX
ENDFIX
"""
println(calcScript)
return calcScript
```
## 8.5	Validate negative value
```bash
operation.grid.dataCellIterator{DataCell cell -> cell.edited}.each { DataCell cell -> 
if ((cell.data < 0))
	cell.addValidationError(0xFF6347, "Cannot enter negative numbers") }
```
## 8.6	 Validate whole number
```bash
operation.grid.dataCellIterator('OWP_Headcount').each {
     if(it.data % 1 != 0 && it.isEdited()){
         it.addValidationError(16745375,'has to be a whole number')
     }
  }
8.7	 Validate e-mail
/*RTPS: {rtpAdminEmail}*/
// Create the message bundle
def mbUs = messageBundle (
 ["EMAIL":"The entered email address ({0}) is not valid. Please recheck email address and try again.",
 ] )
def mbl = messageBundleLoader(["en" : mbUs])
// validate rtpAdminEmail
validateRtp(rtps.rtpAdminEmail, """^([a-zA-Z0-9._%-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,6})*\$""", mbl, "EMAIL", rtps.rtpAdminEmail.EnteredValue)
8.8	Validate if a text field is an already existing Alias
def user = operation.user.getFullName() // Get the full name of the user
// Function to get the alias from member and dimension
String getMemberAlias(String member, String Dimension) {
Cube cube = operation.application.getCube("OEP_PFP")
Dimension dim = operation.application.getDimension(Dimension, cube)
Member mbr = dim.getMember(member, cube)
Map mbrProperties = mbr.toMap()
if(mbrProperties["Alias: Default"] == null){
return (mbr.getName())
	}
else {return (mbr.getAlias("Default"))}
}
// Operation to bring all level zero members from project dimension
Cube cube = operation.application.getCube("OEP_PFP") // Get the cube for "OEP_PF"
Dimension dim = operation.application.getDimension("Project", cube) // Get the Project dimension
List<String> relativeProjects = dim.getEvaluatedMembers('Lvl0Descendants("All Project")', cube)*.name
//Operation to validate the cuurent grid
operation.grid.dataCellIterator('CLD_Move To Annual Plan').each {cell->    
// 1. Identify the projects that were select to move to plan, and that have a budget amount
if(cell.getFormattedValue() == 'Yes'){
    //Validate that the Alias already exist
    def alias = cell.crossDimCell('Name').getFormattedValue()
    relativeProjects.each{ aliasmember-> 
    if( getMemberAlias(aliasmember, "Project") == alias){
    throwVetoException("Dear ${user}, the alias '" + alias + "' already exist, please check the column '" + getMemberAlias(cell.crossDimCell('Name').getMemberName("Account"), "Account") + "' and the row '" + cell.getMemberName("Project") + "'")
     }
    }  
    if( cell.crossDimCell('CLD_Fiscal Year Start').missing || cell.crossDimCell('CLD_Fiscal Year Start').missing || cell.crossDimCell('CLD_Fiscal Year Start').missing){
    throwVetoException("Dear ${user}, the account is required it cannot be missing, please enter it for the row '" + cell.getMemberName("Project") + "'")
    }  }
}
```
# 9	EPM Automate
# 10	Get Variables

## 10.1	Groovy to take data from User variables and run the calc script
```bash
def uvProfitCenter = operation.application.getUserVariable("ProfitCenter").value.name
println("User Profit Center: " + uvProfitCenter)

def uvVersion = operation.application.getUserVariable("OEP_Version").value.name
println("User Version: " + uvVersion)

def uvRevenueCenter = operation.application.getUserVariable("RevenueCenter").value.name
println("User Revenue Center: " + uvRevenueCenter)

String calcScript;
calcScript = '''
SET UPDATECALC OFF;
FIX(
/*Version*/ 		"OEP_Working"
/*Scenario*/		"OEP_Actual",
/*Planning Order*/	"REVSTAGE_Input", "OFS_Direct Input", 
/*Period*/			@Relative("YearTotal", 0),
/*Currency*/		"USD_SFX_Reporting",
/*Profit Center*/	@Relative("'''+uvProfitCenter+'''", 0), 
/*Entity*/	@Relative("'''+uvRevenueCenter+'''", 0) )

	FIX(&OEP_CurYr, &OEP_PriorYr, @RELATIVE("Product", 0))
    	"AC_ASP" = "AC_NET_SALES" / "Revenue_Units_Load";
    ENDFIX
ENDFIX
'''
println calcScript;
return calcScript;    
```
## 10.2	Groovy Script for Currency Conversion:
```bash
The below code creates block for currencies using data copy from USD by taking currency from attribute assigned to the entity and the reporting currencies.
it takes user variable values used in the form to get the member combinations.
def uvProduct = operation.application.getUserVariable("Product").value.name
println("User Product: " + uvProduct)
def uvScenario = operation.application.getUserVariable("OEP_Scenario").value.name
println("User Scenario: " + uvScenario)
def uvVersion = operation.application.getUserVariable("OEP_Version").value.name
println("User Version: " + uvVersion)
def uvRevenueCenter = operation.application.getUserVariable("RevenueCenter").value.name
println("User Revenue Center: " + uvRevenueCenter)
def uvProfitCenter = operation.application.getUserVariable("ProfitCenter").value.name
println("User Profit Center: " + uvProfitCenter)

Cube cube = operation.application.getCube("OEP_FS")
Application app = cube.getApplication()
def entityDim = app.getDimension("Entity", cube)
def attrFunCurrDim=entityDim.getAttributeDimensions().find{it.name == "ATTR_FUNCTIONAL_CURRENCY"}
def revcenterLocalCurrency = entityDim.getMember(uvRevenueCenter, cube).getAttributeValue(attrFunCurrDim)
String revCenterAttribute = revcenterLocalCurrency.toString()
String revCenterFunCurrency = revCenterAttribute.replace("FUN_","")
if(revCenterFunCurrency.length() > 0){
	println("Revenue Center Local Currency: " + revCenterFunCurrency)
}
def varMonths
def varYear
  if(uvScenario == "OEP_Forecast"){
	varMonths = "&OEP_FcstMnth:Dec"
    varYear = "&OEP_FCSTStartYr"
  }
  if(uvScenario == "OEP_Plan"){
	varMonths = "Jan:Dec"
    varYear = "&OEP_PlanStartYr"
  }
println("Year: " + varYear)
def calcScript = new StringBuilder()
calcScript<<""" 
SET AGGMISSG ON;

FIX($varYear, "$uvScenario", "$uvVersion", "$uvRevenueCenter", @RELATIVE("$uvProduct", 0), @RELATIVE("$uvProfitCenter", 0), @RELATIVE("Planning Order", 0), $varMonths)
	FIX(@RELATIVE("Reporting Currencies", 0))
    	CLEARBLOCK ALL;
    ENDFIX
    DATACOPY "USD" TO "USD_SFX_Reporting";
    DATACOPY "USD" TO "USD_AFX_Reporting";
    DATACOPY "USD" TO "USD_PFX_Reporting";
    DATACOPY "USD" TO "USD_CFX_Reporting";
  """
if(revCenterFunCurrency.length() > 0 && revCenterFunCurrency != "USD"){
	calcScript<<""" 
    FIX("$revCenterFunCurrency")
    	CLEARBLOCK ALL;
    ENDFIX
    DATACOPY "USD" TO "$revCenterFunCurrency";
    FIX(@UDA("Account", "CUR_CONVERT"))
    	"$revCenterFunCurrency" = "USD" * "AC_SFX_RATES"->"No Version"->"No Profit Center"->"No Planning Order"->"No Product"->"No Entity"->"$revCenterFunCurrency";
        "USD_PFX_Reporting" = "$revCenterFunCurrency" / "AC_PFX_RATES"->"No Version"->"No Profit Center"->"No Planning Order"->"No Product"->"No Entity"->"$revCenterFunCurrency";
        "USD_CFX_Reporting" = "$revCenterFunCurrency" / "AC_CFX_RATES"->"No Version"->"No Profit Center"->"No Planning Order"->"No Product"->"No Entity"->"$revCenterFunCurrency";
        "USD_AFX_Reporting" = "$revCenterFunCurrency" / "AC_AFX_RATES"->"No Version"->"No Profit Center"->"No Planning Order"->"No Product"->"No Entity"->"$revCenterFunCurrency";
    ENDFIX
"""
}
calcScript<<""" 
ENDFIX
"""
println(calcScript)
return calcScript
```
## 10.3	Groovy Script for updating Substitution Variables
```bash
/*****************************************************
Get Forecast Week
******************************************************/
String newFcstWeek
// Get the current date
def now = Calendar.getInstance()

// Set the first day of the week to Sunday (optional)
now.setFirstDayOfWeek(Calendar.MONDAY)

// Get the week of the year
def weekOfYear = now.get(Calendar.WEEK_OF_YEAR)

if (weekOfYear < 10) {
	newFcstWeek = "WK0" + weekOfYear

} else {
	newFcstWeek = "WK" + weekOfYear
}

println "Forecast week: |$newFcstWeek|"

/*****************************************************
Update Substitution Variables
******************************************************/
/*
"AMC_FcstWeek"	example: WK10
*/

/* ************* Update AMC_FcstWeek substitution variable ************* */
String cubeName = "CLDWKL"
String varName = "AMC_FcstWeek"
String varValue = newFcstWeek
String oldValue = ""

cube = operation.application.getCube(cubeName)
if (cube.hasSubstitutionVariable(varName)) {
	
	oldValue = cube.getSubstitutionVariableValue(varName)
    cube.setSubstitutionVariableValue(varName, varValue)
    println("The Substitution Variable ${varName} in the Cube ${cube.name} has been changed from ${oldValue} to ${varValue}")
	
} else {
	throwVetoException("The Cube ${cube.name} does not have a Substitution Variable called ${varName}")
}
```
# 11	 Calculations
## 11.1	Currency conversion in ASO cube
```bash
String getCrossJoins(List<List<String>> essIntersection) { 
    String crossJoinString
    if (essIntersection.size() > 1)  {    
        crossJoinString = essIntersection[1..-1].inject('{' + essIntersection[0].join(',') + '}') { concat , members -> "CrossJoin(" + concat + ',{' + members.join(',') + '})' } 
    }        
    return crossJoinString
}
List<List<String>> currentPov = [['[USD]'],['[YearTotal].Levels(0).Members'],['[Template]']]
Cube cube = operation.getApplication().getCube('EEFF')
	CustomCalcParameters calcParameters = new CustomCalcParameters()
	calcParameters.pov = getCrossJoins([['[USD]'],['[101010101].Children'],['[No Cebes]'], ['[Total Data].Levels(0).Members'], ['[No Cecos]'], ['[No Custom2]'], ['[G_GRAN_MARIPOSA].Levels(0).Members'], ['[No Materials]'], ['[No ICP]'],['[AOP]'],['[YearTotal].Levels(0).Members'],['[&Anio_Ppto]'],['[Template]']])
    println calcParameters.pov
    calcParameters.sourceRegion = getCrossJoins([['[Local]'],['[Working]'],['[No Segment]'],['[No Partners]'],['[No Account]']])
    calcParameters.roundDigits = 2
    String script = """
			([USD]) := NONEMPTYTUPLE ([Local])
            ([Local]) / ([Working],[No Segment],[No Partners],[No Account]);
		"""
    println script
    calcParameters.script = script
    cube.executeAsoCustomCalculation(calcParameters)
```

# 12	Other
## 12.1	Groovy for run a rule as Admin
```bash 
/*RTPS: {Scenario},{Version}, {Currency}, {Entity}, {FlexDim1}, {FlexDim2}*/
/*	1. Configure Connection (type: Oracle Enterprise Performance Management Cloud). Here is called "Workforce_Connection"
	2. Update Cube name, here is called WKF_PLN
	3. Update Rules and parameters
*/

String varRTPScenario = rtps.Scenario.toString()
String varRTPVersion = rtps.Version.toString()
String varRTPCurrency = rtps.Currency.toString()
String varRTPEntity = rtps.Entity.toString()
String varRTPBU = rtps.FlexDim1.toString()
String varRTPTitle = rtps.FlexDim2.toString()


println "varRTPScenario: ${varRTPScenario.toString()}"
println "varRTPVersion: ${varRTPVersion.toString()}"
println "varRTPCurrency: $varRTPCurrency"
println "varRTPEntity: $varRTPEntity"
println "varRTPBU: $varRTPBU"
println "varRTPTitle: $varRTPTitle"

/* Parameters to wait for WKF_PLN Rules */
boolean ReversePush
HttpResponse<String> jsonResponse
def awaitCompletion(HttpResponse<String> jsonResponse, String connectionName, String operation) {
    final int IN_PROGRESS = -1
 
    // Parse the JSON response to get the status of the operation. Keep polling the WKF_PLN server until the operation completes.
    ReadContext ctx = JsonPath.parse(jsonResponse.body)
    int status = ctx.read('$.status')
    for(long delay = 50; status == IN_PROGRESS; delay = Math.min(1000, delay * 2)) {
        sleep(delay)
        status = getJobStatus(connectionName, (String)ctx.read('$.jobId'))
    }
    println("$operation ${status == 0 || status == -1 ? "successful" : "failed"}.\n")
    return status == 0
}
 
int getJobStatus(String connectionName, String jobId) {
    HttpResponse<String> pingResponse = operation.application.getConnection(connectionName).get("/rest/v3/applications/WKF_PLN/jobs/" + jobId).asString()
    return JsonPath.parse(pingResponse.body).read('$.status')
}
 

 
/* Build the json string to execute the REST job defined in the payload – update connection name and application name */
    jsonResponse = operation.application.getConnection("Workforce_Connection").post('/rest/v3/applications/WKF_PLN/jobs')
    .header("Content-Type", "application/json")
    .body(json([
        "jobType":"Rules", 
        "jobName":"WKF - Groovy SmartPush Reporting Cube", 
        "parameters":["Scenario":"$varRTPScenario","Version":"$varRTPVersion","Currency":"$varRTPCurrency","Entity":"$varRTPEntity","FlexDim1":"$varRTPBU","FlexDim2":"$varRTPTitle"]
        ])
        )
        .asString();
   
	/*println jsonResponse.body*/
    ReversePush = awaitCompletion(jsonResponse, "Workforce_Connection", "Rule ran as an admin")
   
```
