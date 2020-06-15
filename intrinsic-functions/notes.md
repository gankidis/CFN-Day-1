CloudFormation Intrinsic Functions:
---

AWS CloudFormation provides several built-in functions that help you manage your stacks. You can use intrinsic functions in your templates to assign values to properties that are not available until runtime.

Note: Currently, you can use intrinsic functions in resource properties, outputs, metadata attributes, and update policy attributes.

AWS provides below intrinsic functions.

 Fn::Base64
---

Description:

The intrinsic function Fn::Base64 returns the Base64 representation of the input string. This function is typically used to pass encoded data to Amazon EC2 instances by way of the UserData property.

Syntax:
```
YAML: Fn::Base64: valueToEncode
	
JSON: "Fn::Base64": { valueToEncode}
```

Parameters:

valueToEncode: The string value you want to convert to Base64.

Return Value: The original string, in Base64 representation.

Example:
```
{
    "Resources": {
       "MyInstance": {
          "Type": "AWS::EC2::Instance",
          "Properties": {
             "UserData": {
                 "Fn::Base64":{
                     "Fn::Join":["",[
                         " #!/bin/bash -xe\n",
                         "yum update -y\n",
                         "yum install -y\n",
                         "httpd\n",
                         "mariadb-server\n",
                         "php\n"
                     ]]
                 }
                
                }
             
          }
       }
    }
}
```

Fn::Cidr
---

Description: The intrinsic function Fn::Cidr returns an array of CIDR address blocks. The number of CIDR blocks returned is dependent on the count parameter.

Syntax:
```
YAML:

Fn::Cidr:
- ipBlock
- count
- cidrBits
	
JSON:

"Fn::Cidr":["ipBlock","count","cidrBits"]
```



Parameters:

 ipBlock: The user-specified CIDR address block to be split into smaller CIDR blocks.
    
count: The number of CIDRs to generate. Valid range is between 1 and 256.
    
cidrBits: The number of subnet bits for the CIDR. 32-x where x = cidrBits

Return Value: An array of CIDR address blocks.

Fn::FindInMap
---
Description: The intrinsic function Fn::FindInMap returns the value corresponding to keys in a two-level map that is declared in the Mappings section.

```
Syntax:
YAML: 

Fn::FindInMap: [ MapName, TopLevelKey, SecondLevelKey ]

JSON:

{ "Fn::FindInMap" : [ "MapName", "TopLevelKey", "SecondLevelKey"] }
```
Parameters:

MapName: The logical name of a mapping declared in the Mappings section that contains the keys and values.

TopLevelKey: The top-level key name. Its value is a list of key-value pairs.

SecondLevelKey: The second-level key name, which is set to one of the keys from the list assigned to TopLevelKey.

Return Value: The value that is assigned to SecondLevelKey.

Fn::GetAtt
---

Description: The Fn::GetAtt intrinsic function returns the value of an attribute from a resource in the template.

Syntax:
```
YAML:

Fn::GetAtt: [ logicalNameOfResource, attributeName ]

JSON:

{ "Fn::GetAtt" : [ "logicalNameOfResource", "attributeName" ] }

```

Parameters:

logicalNameOfResource: The logical name of the resource that contains the attribute that you want.

attributeName: The name of the resource-specific attribute whose value you want.

Return Value: The attribute value.

Fn::GetAZs
---

Description: The intrinsic function Fn::GetAZs returns an array that lists Availability Zones for a specified region.

Syntax:
```
YAML:

Fn::GetAZs: region

JSON:

{ "Fn::GetAZs" : "region" }
```
Parameters:

region: The name of the region for which you want to get the Availability Zones.

Return Value: The list of Availability Zones for the region.

Fn::ImportValue
---

Description: The intrinsic function Fn::ImportValue returns the value of an output exported by another stack.

Syntax:
```
YAML:

Fn::ImportValue: sharedValueToImport

JSON:

{ "Fn::ImportValue" : sharedValueToImport }
```

Parameters:

sharedValueToImport: The stack output value that you want to import.

Return Value: The stack output value.

Fn::Join
---

Description: The intrinsic function Fn::Join appends a set of values into a single value, separated by the specified delimiter. If a delimiter is the empty string, the set of values are concatenated with no delimiter.

Syntax:
```
YAML: 
Fn::Join: [ delimiter, [ comma-delimited list of values ] ]

JSON:	
{ "Fn::Join" : [ "delimiter", [ comma-delimited list of values ] ] }
```

Parameters:

delimiter: The value you want to occur between fragments.

ListOfValues: The list of values you want to be combined.

Return Value: The combined string.

Fn::Select
---

Description: The intrinsic function Fn::Select returns a single object from a list of objects by index.

Syntax:
```
YAML:
Fn::Select: [ index, listOfObjects ]

JSON:	
{ "Fn::Select" : [ index, listOfObjects ] }
```
Parameters:

index: The index of the object to retrieve. This must be a value from zero to N-1.

listOfObjects: The list of objects to select from. This list must not be null, nor can it have null entries.

Return Value: The selected object.

Fn::Split
---

Description: To split a string into a list of string values so that you can select an element from the resulting string list.

Syntax:
```
YAML:
Fn::Split: [ delimiter, source string ]

JSON:	
{ "Fn::Split" : [ "delimiter", "source string" ] }
```
Parameters:

delimiter: A string value that determines where the source string is divided.
    
source string: The string value that you want to split.

Return Value: A list of string values.

Fn::Sub
---

Description: The intrinsic function Fn::Sub substitutes variables in an input string with values that you specify.

Syntax:
```
YAML:
Fn::Sub:
- String
- { Var1Name: Var1Value, Var2Name: Var2Value }

JSON:	
{ "Fn::Sub" : [ String, { Var1Name: Var1Value, Var2Name: Var2Value } ] }
```
Parameters:

String: Variables (represented as ${MyVarName}) that are substituted with their associated values at runtime.

VarName: The name of a variable that you included in the String parameter.

VarValue: The value that AWS CloudFormation substitutes for the associated variable name at runtime.

Return Value: AWS CloudFormation returns the original string, substituting the values for all of the variables.

Fn::Transform
---

Description: The intrinsic function Fn::Transform specifies a macro to perform custom processing on part of a stack template.

Syntax:
```
YAML: 

Fn::Transform:
Name : macro name
Parameters :
Key : value

JSON:

{ "Fn::Transform" : { "Name" : macro name, "Parameters" : {key : value, ... } } }
```
Parameters:

 Name: The name of the macro you want to perform the processing.

Parameters: The list parameters, specified as key-value pairs, to pass to the macro.

Return Value: The processed template snippet to be included in the processed stack template.

Ref
---

Description: The intrinsic function Ref returns the value of the specified parameter or resource.

Syntax:
```
YAML:

!Ref logicalName

JSON:

{ "Ref" : "logicalName" }
```
Parameters:

logicalName: The logical name of the resource or parameter you want to dereference.

Return Value: The physical ID of the resource or the value of the parameter.
