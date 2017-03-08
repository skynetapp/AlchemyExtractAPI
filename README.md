#### Date: 25-10-2016
#### Description: This document aims to define the Alchemy Extract API coding.


#### The Folder Structure is as follows:
   
   
   Root Directory | Sub Directory | Sub Directory 
------------ | ------------- | -------------
index.php | | |
Global | DBmongo(Mongo Connection),DBmysql(MySQL Connection), AlchemyAPI(Alchemy API Connection)  | 
Lib | Smarty,Common functions,AlchemyAPI | |
Modules | AlchemyExtract | Alchemy Extract Controller, Alchemy Extract Action, Alchemy Extract MySql, Alchemy Extract View, Alchemy Extract DB Mongo|
Views | AlchemyExtract | header.tpl, footer.tpl(Common files), masterList.tpl,detailList.tpl|

#### Code Flows as follows:
   * To insert or get data from DB code flows.. index.php -> Controller -> Action -> MySql.
   * To view the data code flows.. index.php -> Controller -> View.
   
 
#### Step 1:
  Add the created Url and Alchemy Key in the config.php under bluemix2.0 folder.we can get the Alchemy API Key by logging into IBM Bluemix. 
	
**_Code:_**
	
```
	
$GLOBALS['alchemy_apiKey']='xxxxxxxxxxxxxxxxxxxxxxxxxx';
$GLOBALS['alchemy_url']='https://gateway-a.watsonplatform.net/calls';
	
```
	
  
#### Step 2:
  Create a module name as 'alchemyExtract' in index.php and respective actions will be performed accordingly.
  
#### Step 3:
   In the server normally we will be able to see existing Master list data by running the url -  http://159.203.239.91/bluemix2.0/index.php?module=alchemyExtract&action=GetMasterList. When we click on the update button **GetList** action will be performed from index page and function **getDataFromAlchemyDB()** will be called.
   
#### Step 4:
   From index.php, **AlchemyExtractController** class will be called which controlles all the operations of Alchemy Extract module. Here function **getDataFromAlchemyDB()** will be executed.
   
#### Step 5:
   This **getTextData()** function gets the multiple records data from Request table and sends request to Alchemy API response function **entities('text', $text, null)** one by one using for loop.
   
#### Step 6:
   On receiving response from API, the JSON response will be stored in Mongo DB by calling function  **insertBlueMixJSONResponseIntoMongo(json_encode($data))**.

#### Step 7:
   The response array will be inserted into Master data by function **insertExtractedMasterDataIntoMySQL($data,$id);** from Controller.
   

#### Step 8:
   Here based on the master request id, the response will be stored in Child table by function **getParsedDataFromJSONResponse($masterReqId)**.

#### Step 9:
   On inserting the JSON response into master and child tables, the status and Request date will be updated for that record in the Request table by function **updateTextData($id)** in controller.


#### Step 10:
   To get the Master list function **getALLMasterDataFromMySQL** will be called from controller.
   
#### Step 11:
   To view the Master list function **showExtractedMasterListView()** will be called from controller to View.
   
**_Code:_**

```
  function showExtractedMasterListView($data_arr){
        $smarty = new Smarty();
        $smarty->assign('base_path',$GLOBALS['base_path']);
		$smarty->assign('cursor',$data_arr);
		
	    $smarty->display(''.$GLOBALS['root_path'].'/Views/AlchemyExtract/allMasterList.tpl');
    }
    
``` 

#### Step 12:
   To view the Child data based on the master id, function **getExtractChildDataFromMySQL($post_data)** will be called from controller.
   
   - Function **getAllChildDataFromMySQL($post_data)** will get the records based on the master id using MySql query. 
   - Function **showChildDetailListView($alchemy_list_vo)** will be called in view. 
   
#### Assumptions

- DBMongo - Inserts the Alchemy JSON response into mongo. It is included in Global -> DBMongo.

#### Errors

If any erros found, the following may be the reasons.

- Url might not be correct. It should be as below.

**_Url:_**

```
http://159.203.239.91/bluemix2.0/index.php?module=alchemyExtract&action=GetMasterList

```
- The root path, base path and the database name should be correct in config.php.
- The alchemy API key should be a valid key from IBM Bluemix. If the key is expired, the results may not be appeared.
- If the day limit is exceeded for the API key, then also the child results will not be inserted. 


#### MySQL Database Details

  
 Database Name: bluemix
 
 Tables | Description | Fields |
------------ | ------------- | ----------
BlueMixAlmEntityExtractReq | Request table to get records where EntityExtractionStatus='' and status=0 | |
alchemy_master | Stores the extracted data based on the request id | alm_id, alm_date, alm_external_id, alm_response_text |
alchemy_child | Stores the child records based on master id | alc_id, alc_master_id, alc_type, alc_relevance, alc_count, alc_text |
 
 
 #### Mongo Database details
 
 Database Name: lytepole
 
 



   
