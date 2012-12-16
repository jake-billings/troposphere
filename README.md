troposphere - library to create AWS CloudFormation descriptions

The troposphere library allows for easier creation of the AWS CloudFormation
JSON by writing Python code to describe the AWS resources. To facilitate
catching CloudFormation or JSON errors early the library has property and type
checking built into the classes. Examples of the error checking
(full tracebacks removed for clarity):

Incorrect property being set on AWS resource:
```
>>> import troposphere.ec2 as ec2
>>> ec2.Instance("ec2instance", image="i-XXXX")
Traceback (most recent call last):
...
AttributeError: AWS::EC2::Instance object does not support attribute image
```

Incorrect type for AWS resource property:
```
>>> ec2.Instance("ec2instance", ImageId=1)
Traceback (most recent call last):
...
TypeError: ImageId is <type 'int'>, expected <type 'basestring'>
```

Missing required property for the AWS resource:
```
>>> from troposphere import Template
>>> import troposphere.ec2 as ec2
>>> t = Template()
>>> t.add_resource(ec2.Instance("ec2instance", ImageId="ami-3bcc9e7e"))
<troposphere.ec2.Instance object at 0x109ee2e50>
>>> print t.to_json()
Traceback (most recent call last):
...
ValueError: Resource InstanceType required in type AWS::EC2::Instance
```

Currently supported AWS resource types:
- AWS::AutoScaling
- AWS::EC2
- AWS::ElasticLoadBalancing

Todo:
- Add additional validity checks
- Don't allow AWS helper override for list typed objects
- Add missing AWS resource types:
  - AWS::CloudFormation
  - AWS::CloudFront
  - AWS::CloudWatch
  - AWS::ElastiCache
  - AWS::ElasticBeanstalk
  - AWS::IAM
  - AWS::RDS
  - AWS::Route53
  - AWS::S3
  - AWS::SDB

Duplicating a single instance sample would look like this:

```
# Converted from EC2InstanceSample.template located at:
# http://aws.amazon.com/cloudformation/aws-cloudformation-templates/

from troposphere import Base64, FindInMap, GetAtt
from troposphere import Parameter, Output, Ref, Template
import troposphere.ec2 as ec2


template = Template()

keyname_param = template.add_parameter(Parameter("KeyName",
    Description="Name of an existing EC2 KeyPair to enable SSH "
                "access to the instance",
    Type="String",
))

template.add_mapping('RegionMap', {
    "us-east-1":      {"AMI": "ami-7f418316"},
    "us-west-1":      {"AMI": "ami-951945d0"},
    "us-west-2":      {"AMI": "ami-16fd7026"},
    "eu-west-1":      {"AMI": "ami-24506250"},
    "sa-east-1":      {"AMI": "ami-3e3be423"},
    "ap-southeast-1": {"AMI": "ami-74dda626"},
    "ap-northeast-1": {"AMI": "ami-dcfa4edd"}
})

ec2_instance = template.add_resource(ec2.Instance("Ec2Instance",
    ImageId=FindInMap("RegionMap", Ref("AWS::Region"), "AMI"),
    InstanceType="t1.micro",
    KeyName=Ref(keyname_param),
    SecurityGroups=["default"],
    UserData=Base64("80")
))

template.add_output([
    Output("InstanceId",
        Description="InstanceId of the newly created EC2 instance",
        Value=Ref(ec2_instance),
    ),
    Output("AZ",
        Description="Availability Zone of the newly created EC2 instance",
        Value=GetAtt(ec2_instance, "AvailabilityZone"),
    ),
    Output("PublicIP",
        Description="Public IP address of the newly created EC2 instance",
        Value=GetAtt(ec2_instance, "PublicIp"),
    ),
    Output("PrivateIP",
        Description="Private IP address of the newly created EC2 instance",
        Value=GetAtt(ec2_instance, "PrivateIp"),
    ),
    Output("PublicDNS",
        Description="Public DNSName of the newly created EC2 instance",
        Value=GetAtt(ec2_instance, "PublicDnsName"),
    ),
    Output("PrivateDNS",
        Description="Private DNSName of the newly created EC2 instance",
        Value=GetAtt(ec2_instance, "PrivateDnsName"),
    ),
])

print template.to_json()
```