### network_gcp_ssl_enforce.sentinel
```
GCP_SSL_ENFORCE: As per policy, only https access will be allowed.
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

#### Maps
|Name|Description|
|----|-----|
|resourceTypesSSLEnforceMap|This is the map, being used to have path of nodes for the respective gcp service for SSL Enforcement policy.|

#### Methods
The below function is being used to validate the value of parameter "enable_http_port_access". As per the policy, its value can not be true. If the policy will not be validated successfully, it will generate appropriate message to show the users. This function will have below 2-parameters:

* Parameters

  |Name|Description|
  |----|-----|
  |address|The key inside of resource_changes section for particular GCP Resource in tfplan mock.|
  |rc|The value of address key inside of resource_changes section for particular GCP Resource in tfplan mock.|

  ```
  check_endpoint_config = func(address, rc) {

	key = resourceTypesSSLEnforceMap[rc.type]["key"]
	selected_node = plan.evaluate_attribute(rc, key)
	
	if  selected_node {
		return "Http port's access needs to be disabled for the " + plan.to_string(address) + " services, please set value false to make it disabled"
	} else {
		return null
	}
  }
  ```

#### Working Code
```
messages_http = {}

for resourceTypesSSLEnforceMap as key_address, _ {
	
	# Get all the instances on the basis of type
	allResources = plan.find_resources(key_address)
	
	for allResources as address, rc {
		message = null
		message = check_endpoint_config(address, rc)

		if types.type_of(message) is not "null"{
			
			gen.create_sub_main_key_list( messages, messages_http, address)

			append(messages_http[address],message)
			append(messages[address],message)

		}
	}
}
```

#### Main Rule
The main function returns true/false as per value of GCP_SSL_ENFORCE 
```
GCP_SSL_ENFORCE = rule {
	length(messages_http) is 0
}

main = rule { GCP_SSL_ENFORCE }