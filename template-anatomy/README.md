Template Anatomy 
---

AWSTemplateFormatVersion: 'version date' (optional) # version of the CloudFormation template. Only accepted value is '2010-09-09'

Description: 'String' (optional) # a text description of the Cloudformation template

Metadata: 'template metadata' (optional) # objects that provide additional information about the template

Parameters: 'set of parameters' (optional) # a set of inputs used to customize the template

Mappings: 'set of mappings' (optional) # a mapping of keys and associated values

Conditions: 'set of conditions' (optional) # conditions that control whether certain resources are created

Transform: 'set of transforms' (optional) # for serverless applications

Resources: 'set of resources' (required) # a components of your infrastructure

Outputs: 'set of outputs' (optional) # values that are returned whenever you view your stack's properties
