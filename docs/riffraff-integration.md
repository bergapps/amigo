## Amigo - Riffraff integration

In order to have up to date instances running our services we can use Amigo and Riffraff (their forces combined!).
Namely Amigo creates AMIs in a scheduled way, using everytime the latest versions of the dependencies we have declared 
(e.g. each week Amigo will create an AMI with the latest version of Java 8).


### Update the Cloudformation script
In order to use those up to date images, we have to have as a parameter in our cloudformation scripts the AMI id we want to use.
For example (in json):
```json
...
    "AMI": {
      "Description": "AMI to use for instances",
      "Type": "AWS::EC2::Image::Id"
    }
```
This is the only thing that is required in the cloudformation script. The name of the parameter can be anything (ideally with a meaningful name!), but "AMI" is the default expected one.


### Update the Riffraff file (riff-raff.yaml or deploy.json)
After doing that we would like Riffraff to check in every deployment whether there is a a newer AMI than the one currently used. In order to do that, RiffRaff should know for which image we are talking about (eg. Ubuntu Xenial with Java 8) and for which cloudformation script.
 -	In order to define the AMI: We must define its tags in our `riff-raff.yaml` (or `deploy.json`) file. This can be done either using the `amiTags` parameter or the `amiParametersToTags` parameter, if we use multiple AMIs in one cloudformation script or if we want to define explicitly the name of the AMI parameter. 
 -	In order to define the cloudformation script we can either define it using tags or its name. When using tags we find the stack 	by looking for one with matching stack, app and stage tags in the same way that autoscaling groups are discovered. When using the cloudformation name we set the parameter `cloudFormationStackName`. To set which of the two ways to use, we must set the parameter `cloudFormationStackByTags`.

For more detailed description of the parameters please check here: 
https://riffraff.gutools.co.uk/docs/magenta-lib/types#amicloudformationparameter

Example:
```yaml
[riff-raff.yaml]
  update-ami:
    type: ami-cloudformation-parameter
    parameters:
      cloudFormationStackByTags: false
      prependStackToCloudFormationStackName: false
      cloudFormationStackName: cloudformation-stack-name
      amiParametersToTags:
        AMI:
          Recipe: ubuntu-wily-java8
          BuiltBy: amigo
          AmigoStage: PROD 
```

### How does Riffraff find the AMIs that correspond to certain tags
Riffraff uses [Prism](https://github.com/guardian/prism).
Prism has a rich and expressive rest API that can help us understand which AMI will be used, for the tags we have set in our `riff-raff.yaml`.

E.g. in the previous example we have the tags: `Recipe: ubuntu-wily-java8`, `BuiltBy: amigo`, `AmigoStage: PROD`.
The request we can make to Prism is:
http://prism.gutools.co.uk/images?tags.BuiltBy=amigo&tags.AmigoStage=PROD&tags.Recipe=ubuntu-wily-java8

The query will return a list of AMIs. 
From that list, Riffraff will find the most recently created (or "baked" in Amigo's terminology) AMI and use that.



