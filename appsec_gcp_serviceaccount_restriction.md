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
The below map is having entries of the GCP resources in key/value pair, those are required to be validated for Service Account policy. Key will be name of the GCP terraform resource ("https://registry.terraform.io/providers/hashicorp/google/latest/docs") and its value will be again combination of key/value pair. Here now key will be ```key``` only and value will be the path of service account node. Since this is the generic one and can validate service account associated with any google resource. In order to validate, just need to add corresponding entry of particular GCP terraform resource with the path of its service account in the below map as given for ```google_compute_instance``` or ```google_dataproc_cluster``` or ```example_rsc```. 
```
resourceTypesServiceAccountMap = {
	"google_compute_instance" : {
		"key":   "service_account.0.email",
	},
	"google_dataproc_cluster": {
		"key":   "cluster_config.0.gce_cluster_config.0.service_account",
	},
	"example_rsc": {
	     "key": "someroot.service_account",
	},
}
```

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

#### Working Code
The below code will iterate each member of resourceTypesServiceAccountMap, which will belong to any resource eg. google_compute_instance/google_dataproc_cluster etc and each member will have path of its service account as value. The code will evaluate the service account's information by using this value and validate the said policy. 

```
messages_sa = {}
for resourceTypesServiceAccountMap as key_address, _ {
	# Get all the instances on the basis of type
	allResources = plan.find_resources(key_address)
	for allResources as address, rc {

		message = null		
		message = check_service_account_config(address, rc)
		if types.type_of(message) is not "null" {
		
			gen.create_sub_main_key_list(messages, messages_sa, address)
			
			append(messages_sa[address],message)
			append(messages[address],message)
		}
	}
}
```

#### Main Rule
The main function returns true/false as per value of SVC_ACCOUNT_CHECK 
```
SVC_ACCOUNT_CHECK = rule {
  	length(messages_sa) is 0 
}

main = rule { SVC_ACCOUNT_CHECK }
```
