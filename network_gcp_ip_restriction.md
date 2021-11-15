### network_gcp_ip_restriction.sentinel
```
GCP_INTERNAL_IP: As per policy, Internal IPs will be enabled only, no communication thorugh external ip addresses.
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
|messages| It is being used to hold the complete message of policies violation to show to the user.|

#### Maps
The below map is having entries of the GCP resources in key/value pair, those are required to be validated for ip restriction policy. Key will be name of the GCP terraform resource ("https://registry.terraform.io/providers/hashicorp/google/latest/docs") and its value will be again combination of key/value pair. Here now key will be ```key``` only and value will be the path of internal_ip_only node. Since this is the generic one and can validate internal ip associated with any google resource. In order to validate, just need to add corresponding entry of particular GCP terraform resource with the path of its internal_ip_only in the below map as given for google_dataproc_cluster or example_rsc.
```
resourceTypesInternalIPMap = {	
	"google_dataproc_cluster": {
		"key":   "cluster_config.0.gce_cluster_config.0.internal_ip_only",
	},
    "example_rsc": {
	     "key": "someroot.internal_ip_only",
	},
}
```

#### Methods
The below function is being used to validate the value of parameter ```internal_ip_only``` As per the policy, its value needs to be true and it can not be empty/null. If the policy won't be validated successfully, it will generate appropriate message to show the users. This function will have below 2-parameters:

* Parameters

  |Name|Description|
  |----|-----|
  |address|The key inside of resource_changes section for particular GCP Resource in tfplan mock.|
  |rc|The value of address key inside of resource_changes section for particular GCP Resource in tfplan mock.|

  ```
  check_internal_ip = func(address, rc) {

	key = resourceTypesInternalIPMap[rc.type]["key"]
	selected_node = plan.evaluate_attribute(rc, key)

	if types.type_of(selected_node) is "null" {
		return plan.to_string(address) + " does not have " + key +" defined"
	} else {
		if not selected_node {					
			return plan.to_string(address) +  " service will be accessible through internal ip only but it is disabled here, please set value true to make it   enable"			
		} else {
			return null
		}
	}
  }
  ```

#### Working Code
The below code will iterate each member of resourceTypesInternalIPMap, which will belong to any resource eg. google_dataproc_cluster etc and each member will have path of its internal_ip_only as value. The code will evaluate the internal_ip_only's information by using this value and validate the said policy.
```
messages_ip_internal = {}

for resourceTypesInternalIPMap as key_address, _ {
	# Get all the instances on the basis of type
	allResources = plan.find_resources(key_address)
	for allResources as address, rc {
		message = null
		message = check_internal_ip(address, rc)

		if types.type_of(message) is not "null" {

			gen.create_sub_main_key_list(messages, messages_ip_internal, address)
			
			append(messages_ip_internal[address],message)
			append(messages[address],message)
		} 	
	}
}
```

#### Main Rule
The main function returns true/false as per value of GCP_INTERNAL_IP 
```
GCP_INTERNAL_IP = rule {
 	length(messages_ip_internal) is 0 
}

main = rule { GCP_INTERNAL_IP }
```
