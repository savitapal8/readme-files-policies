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
|Name|Description|
|----|-----|
|resourceTypesInternalIPMap|This is the map, being used to have path of nodes for the respective gcp service for Internal IP policy.|

#### Methods
The below function is being used to validate the value of parameter "internal_ip_only". As per the policy, its value needs to be true and it can not be empty/null. If the policy won't be validated successfully, it will generate appropriate message to show the users. This function will have below 2-parameters:

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