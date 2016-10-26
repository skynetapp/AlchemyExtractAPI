#### Date: 25-10-2016
#### Description: This document aims to define the Alchemy Extract API coding 


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
  Add the created Url and Alchemy Key in the config.php under bluemix2.0 folder.
	
**_Code:_**
	
```
	
$GLOBALS['alchemy_apiKey']='75c6700778eda02ffce454aed26b51f1faad5f54';
$GLOBALS['alchemy_url']='https://gateway-a.watsonplatform.net/calls';
	
```
	
  
#### Step 2:
  Create a module name as 'alchemyExtract' in index.php and respective actions will be performed accordingly.
  
**_Code:_**

```
<?php
if($_REQUEST['module']=='alchemyExtract'){ 
    switch ($_REQUEST['action']){
        case 'GetList':
        {
			 
            $alchemyController = AlchemyExtractController::getInstance();
            $alchemyController->getDataFromAlchemyDB();
            break;
        }
	case 'GetMasterList':
        {
			 
            $alchemyController = AlchemyExtractController::getInstance();
            $alchemyController->getMasterListData();
            break;
        }
        case 'DetailList':
        {
            $alchemyController = AlchemyExtractController::getInstance();
            $alchemyController->getExtractChildDataFromMySQL($_REQUEST);
            break;
        }
      
    }
}
?>

```

#### Step 3:
   In the server normally we will be able to see existing Master list data. When we click on the update button **GetList** action will be performed from index page and **getDataFromAlchemyDB()** will be called.
   
#### Step 4:
   From index.php, **AlchemyExtractController** class will be called which controlles all the operations of Alchemy Extract module. Here function **getDataFromAlchemyDB()** will be executed.
   
#### Step 5:
   This **getTextData()** function gets the multiple records data from Request table and sends request to Alchemy API response function **entities('text', $text, null)** one by one using for loop.
   
#### Step 6:
   On receiving response from API, the JSON response will be stored in Mongo DB by calling function  **insertBlueMixJSONResponseIntoMongo(json_encode($data))**.

#### Step 7:
   The response array will be inserted into Master data by function **insertExtractedMasterDataIntoMySQL($data,$id);** from Controller.
   
**_Code:_**

```  
 public function insertAllExtractedMasterDataIntoMySQL($data,$id){
		$this->response_array =$data;
		$masterId=$this->insertIntoMasterData(json_encode($data),$id);
        if($masterId>0)
		return $this->getParsedDataFromJSONResponse($masterId);
	}
	
	
	public function insertIntoMasterData($json_response,$id){
    	$sql = "INSERT INTO alchemy_master
               (
                alm_sugar_id,
                alm_response_text,
                alm_date,alm_external_id
                ) VALUES (
                
                '',
                '$json_response',
                NOW(),
				'$id'
                )";
		mysqli_query($this->con,$sql);
        return $this->con->insert_id;
    }

```

#### Step 8:
   Here based on the master request id, the response will be stored in Child table by function **getParsedDataFromJSONResponse($masterReqId)**.

**_Code:_**

```
   public function getParsedDataFromJSONResponse($masterId){
    	foreach($this->response_array['entities'] as $entityKey=>$entityVal) {
            $row['alc_master_id']=$masterId;
            $row['alc_type']=$entityVal['type'];
            $row['alc_relevance']=$entityVal['relevance'];
            $row['alc_count']=$entityVal['count'];
            $row['alc_text']=$entityVal['text'];
            //$row['alc_quotations_statement']=$entityVal['statement'];
            $row['alc_knowledgegraph_typehierarchy']=$entityVal['knowledgegraph']['typehierarchy'];
            $row['alc_sentiment_type']=$entityVal['sentiment']['type'];
            $row['alc_sentiment_score']=$entityVal['sentiment']['score'];
            $row['alc_sentiment_mixed']=$entityVal['sentiment']['mixed'];
            $row['alc_disambiguated_name']=$entityVal['disambiguated']['name'];

            $row['alc_disambiguated_website']=$entityVal['disambiguated']['website'];
            $row['alc_disambiguated_geo']=$entityVal['disambiguated']['geo'];
            $row['alc_disambiguated_dbpedia']=$entityVal['disambiguated']['dbpedia'];
            $row['alc_disambiguated_yago']=$entityVal['disambiguated']['yago'];
            $row['alc_disambiguated_opencyc']=$entityVal['disambiguated']['opencyc'];
            $row['alc_disambiguated_umbel']=$entityVal['disambiguated']['umbel'];
            $row['alc_disambiguated_freeebase']=$entityVal['disambiguated']['freeebase'];
            $row['alc_disambiguated_ciafactbook']=$entityVal['disambiguated']['ciafactbook'];
            $row['alc_disambiguated_census']=$entityVal['disambiguated']['census'];
            $row['alc_disambiguated_geonames']=$entityVal['disambiguated']['geonames'];
            $row['alc_disambiguated_musicbrainz']=$entityVal['disambiguated']['musicbrainz'];
            $row['alc_disambiguated_crunchbase']=$entityVal['disambiguated']['crunchbase'];
            /*$row['alc_subtype_id']=$entityVal[''];
            $row['alc_subtype_name']=$entityVal[''];*/
            $row['alc_quotation']=$entityVal['quotation'];

            if(count($entityVal['disambiguated']['subtype'])>0)
            foreach($entityVal['disambiguated']['subtype'] as $disStypeKey => $disStypeVal){
                $row['alc_disambiguated_subtype']=$disStypeVal;
                $this->insertIntoRowMysqlTable($row);
            }
            else $this->insertIntoRowMysqlTable($row);
        }
	    return true;
    }

    public function insertIntoRowMysqlTable($rowData=array()){
    	$sql = "INSERT INTO alchemy_child(
                    alc_master_id,
                    alc_type,
                    alc_relevance,
                    alc_count,
                    alc_text,
                    alc_quotations_statement,
                    alc_knowledgegraph_typehierarchy,
                    alc_sentiment_type,
                    alc_sentiment_score,
                    alc_sentiment_mixed,
                    alc_disambiguated_name,
                    alc_disambiguated_subtype,
                    alc_disambiguated_website,
                    alc_disambiguated_geo,
                    alc_disambiguated_dbpedia,
                    alc_disambiguated_yago,
                    alc_disambiguated_opencyc,
                    alc_disambiguated_umbel,
                    alc_disambiguated_freeebase,
                    alc_disambiguated_ciafactbook,
                    alc_disambiguated_census,
                    alc_disambiguated_geonames,
                    alc_disambiguated_musicbrainz,
                    alc_disambiguated_crunchbase,
                    alc_subtype_id,
                    alc_subtype_name,
                    alc_quotation,
                    alc_date
                ) VALUES (
                    '".$rowData['alc_master_id']."',
                    '".$rowData['alc_type']."',
                    '".$rowData['alc_relevance']."',
                    '".$rowData['alc_count']."',
                    '".$rowData['alc_text']."',
                    '".$rowData['alc_quotations_statement']."',
                    '".$rowData['alc_knowledgegraph_typehierarchy']."',
                    '".$rowData['alc_sentiment_type']."',
                    '".$rowData['alc_sentiment_score']."',
                    '".$rowData['alc_sentiment_mixed']."',
                    '".$rowData['alc_disambiguated_name']."',
                    '".$rowData['alc_disambiguated_subtype']."',
                    '".$rowData['alc_disambiguated_website']."',
                    '".$rowData['alc_disambiguated_geo']."',
                    '".$rowData['alc_disambiguated_dbpedia']."',
                    '".$rowData['alc_disambiguated_yago']."',
                    '".$rowData['alc_disambiguated_opencyc']."',
                    '".$rowData['alc_disambiguated_umbel']."',
                    '".$rowData['alc_disambiguated_freeebase']."',
                    '".$rowData['alc_disambiguated_ciafactbook']."',
                    '".$rowData['alc_disambiguated_census']."',
                    '".$rowData['alc_disambiguated_geonames']."',
                    '".$rowData['alc_disambiguated_musicbrainz']."',
                    '".$rowData['alc_disambiguated_crunchbase']."',
                    '".$rowData['alc_subtype_id']."',
                    '".$rowData['alc_subtype_name']."',
                    '".$rowData['alc_quotation']."',
                    NOW()
                )";
        mysqli_query($this->con,$sql);
        return $this->con->insert_id;
    }
    
```

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
   Function **getAllChildDataFromMySQL($post_data)** will get the records based on the master id using MySql query. Function **showChildDetailListView($alchemy_list_vo)** will be called in view. 
   
**_Code:_**

```
public function getExtractChildDataFromMySQL($post_data){
        $alchemy_action = AlchemyExtractAction::getInstance();
        $alchemy_list_vo = $alchemy_action->getAllChildDataFromMySQL($post_data);
		$alchemy_view = AlchemyExtractView::getInstance();
    	$alchemy_view->showChildDetailListView($alchemy_list_vo);
	}
```
