### appsec_gcp_serviceaccount_restriction.sentinel
```
SVC_ACCOUNT_CHECK: As per policy, service account must be custom.
```

#### Imports
```
import "strings"
import "types"
import "tfplan-functions" as plan
import "generic-functions" as gen
```

#### Variables
|Name|Description|
|----|-----|
|selected_node|It is being used locally to have information of node by passing the path.|
|messages|It is being used to hold the complete message of policies violation to show to the user.|
|default_compute_sa|This is having email id of default compute service account to validate.|

#### Maps
|Name|Description|
|----|-----|
|resourceTypesServiceAccountMap|This is the map, being used to have path of node for the respective gcp service for Service Account policy. Here Key is having complete path of particular node.|

#### Methods
The below function is being used to validate the value of parameter "service_account". As per the policy, SA can not be empty/null otherwise it will take default compute service account later on. There must be either reference of service account resource or any email. If the policy won't be validated successfully, it will generate appropriate message to show the users. This function will have below 2-parameters:

* Parameters

  |Name|Description|
  |----|-----|
  |address|The key inside of resource_changes section for particular GCP Resource in tfplan mock|
  |rc|The value of address key inside of resource_changes section for particular GCP Resource in tfplan mock|


  ```
  check_service_account_config = func(address, rc) {	

	key = resourceTypesServiceAccountMap[rc.type]["key"]
	selected_node = plan.evaluate_attribute(rc, key)
	
	if types.type_of(selected_node) is "null" {					
		result = plan.evaluate_attribute(rc.change.after_unknown, key)

		if result is not "null" and result is not true {
			return address + " service is not having any service account, please assign it"			
		} else {
			return null
		}
	
	} else {
			if types.type_of(selected_node) is "null" {
				return  address + " service is not having any service account, please assign it"
			} else {
					arr_sa = strings.split(selected_node, default_compute_sa)
					
					if length(arr_sa) > 1 {
						return "The service account of " + address + " service can not be a default compute service account, please change it"						
					} else {
						return null
					}
			}	
	}	
  }
  ```