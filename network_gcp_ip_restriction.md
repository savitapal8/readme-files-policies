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
The below map is having entries of the GCP resources in key/value pair, those are required to be validated for ip restriction policy. Key will be name of the GCP terraform resource ("https://registry.terraform.io/providers/hashicorp/google/latest/docs") and its value will be again combination of key/value pair. Here now key will be ```key``` only and value will be the path of ```internal_ip_only node for google_dataproc_cluster``` and ```ipv4_enabled node for google_sql_database_instance```. Since this is the generic one and can validate internal ips access associated with any google resource. In order to validate, just need to add corresponding entry of particular GCP terraform resource with the path of its respective node in the below map as given for google_dataproc_cluster or google_sql_database_instance and ```expected_result``` will have expected value for particular google resource.
```
resourceTypesInternalIPMap = {	
	"google_dataproc_cluster": {
		"key": "cluster_config.0.gce_cluster_config.0.internal_ip_only",
		"expected_result" : true,
	},
	"google_sql_database_instance": {
		"key": "settings.0.ip_configuration.0.ipv4_enabled",
		"expected_result" : false,
	},
    	"example_rsc": {
	     "key": "path of node",
	     "expected_result" : whatever expected as per google resource(true/false),
	},
}
```

#### Methods
The below function is being used to validate the value of parameter ```internal_ip_only/ipv4_enabled``` As per the policy, its value needs to be true/false as per googel resource and it can not be empty/null. If the policy won't be validated successfully, it will generate appropriate message to show the users. This function will have below 2-parameters:

* Parameters

  |Name|Description|
  |----|-----|
  |address|The key inside of resource_changes section for particular GCP Resource in tfplan mock.|
  |rc|The value of address key inside of resource_changes section for particular GCP Resource in tfplan mock.|

  ```
  check_internal_ip = func(address, rc) {

	key = resourceTypesInternalIPMap[rc.type]["key"]
	selected_node = plan.evaluate_attribute(rc, key)

	selected_node_result = resourceTypesInternalIPMap[rc.type]["expected_result"]

	if types.type_of(selected_node) is "null" or types.type_of(selected_node) is "undefined" {
		return plan.to_string(address) + " does not have " + key + " defined"
	} else {
		if selected_node is selected_node_result {
			return null
		} else {	
			return plan.to_string(address) + " service will be accessible through internal ip only, please set value " + 								plan.to_string(selected_node_result) + " to make it as per requirement."			
		}
	}
  }
  ```

#### Working Code
The below code will iterate each member of resourceTypesInternalIPMap, which will belong to any resource eg. google_dataproc_cluster or google_sql_database_instance etc and each member will have path of its internal_ip_only or ipv4_enabled as value. The code will evaluate the internal_ip_only or ipv4_enabled information by using this value and validate the said policy.


#### Main Rule
The main function returns true/false as per value of GCP_INTERNAL_IP 
```
GCP_INTERNAL_IP = rule {
 	length(messages_ip_internal) is 0 
}

# Main rule
print(messages)

main = rule { GCP_INTERNAL_IP }
```
