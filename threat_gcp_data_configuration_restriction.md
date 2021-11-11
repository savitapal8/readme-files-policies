### threat_gcp_data_configuration_restriction.sentinel
* **GCP_DLP_TRIGGER:** Enforce jobs to have a trigger to ensure they run automatically.
* **GCP_DLP_SAVEFINDINGS:** Enforce that all jobs save their findings into a BQ dataset.

#### Variables 
|Name|Description|
|----|-----|
|selected_node|It is being used locally to have information of node by passing the path.|
|messages|It is being used to hold the complete message of policies violation to show to the user.|

#### Maps
|Name|Description|
|----|-----|
|resourceTypesDLPJTMap|This is the map, being used to have path of node for the respective gcp service for Job  Trigger policy. Here Key is having complete path of particular node.|
|resourceTypesDLPSFMap|This is the map, being used to have path of nodes for the respective gcp service for Save Finding policy. It is having two enteries with two keys "key" and "inspect_key". As per the policy, "inspect_job.0.actions.0.save_findings.0.output_config.0.table.0.dataset_id" needs to have appropriate value for the dataset_id. As per terraform, inspect_job is not required section, so "inspect_job" & "dataset_id" both need to be validated for null/empty.|

#### Methods
The below function is being used to validate the value of parameter "recurrence_period_duration". As per the policy, its value needs to be lied between 1 day to 60 days. It can not be empty/null and will be sufficed with 's'. If the policy won't be validated successfully, it will generate appropriate message to show the users. This function will have below 2-parameters:

* Parameters
  |Name|Description|
  |----|-----|
  |address|The key inside of resource_changes section for particular GCP Resource in tfplan mock|
  |rc|The value of address key inside of resource_changes section for particular GCP Resource in tfplan mock|
      
```
      check_job_trigger = func(address, rc) {

	    key = resourceTypesDLPJTMap[rc.type]["key"]
	
	    selected_node = plan.evaluate_attribute(rc, key)
	
	    if types.type_of(selected_node) is "null" {
		    return plan.to_string(address) + " does not have " + key +" defined"
	    } else {
		
		 result = strings.has_suffix(selected_node, "s")

		 min_val = 86400
		 max_val = 86400 * 60
		
		 if selected_node is not "" and result is true {
		   	str_value = strings.split(selected_node,"s")[0]
			
			if float(str_value) >= float(min_val) and float(str_value) <= float(max_val) {
				return null 
			} else {
				return plan.to_string(address) +  " recurrence_period_duration must be set to a time duration greater than or equal to 1 day and can be no longer than 60 days."							
			}
		 } else {
			return plan.to_string(address) +  " recurrence_period_duration is not having valid input, please provide correctly"				
		}
	  }
    }

```

The below function is being used to validate the value of parameter "inspect_job.0.actions.0.save_findings.0.output_config.0.table.0.dataset_id". As per the policy, its value can not be null/empty and must be proper valid dataset_id. If the policy won't be validated successfully, it will generate appropriate message to show the users. This function will have below 2-parameters:

* Parameters
  |Name|Description|
  |----|-----|
  |address|The key inside of resource_changes section for particular GCP Resource in tfplan mock|
  |rc|The value of address key inside of resource_changes section for particular GCP Resource in tfplan mock|

```
      check_save_findings = func(address, rc) {

	     key = resourceTypesDLPSFMap[rc.type]["inspect_key"]
	     selected_node = plan.evaluate_attribute(rc, key)
	
	     if selected_node is [] {
		     return plan.to_string(address) + " does not have " + key +" defined"
	     } else {

		  key = resourceTypesDLPSFMap[rc.type]["key"]
		  selected_node = plan.evaluate_attribute(rc, key)
		
		  if selected_node is not ""{
			  return null
		  } else {
			   return plan.to_string(address) +  " dataset id is not having valid input, please provide correctly"			
		  }
	   }
    }
```