# Notes
## AWS 
### MFA login

<p style="color:red;">S3UploadFailedError: Failed to upload test.gz to webhose100/test.gz: An error occurred (AccessDenied) when calling the PutObject operation: Access Denied</p>

#### Reason
Account have MFA enabled. 

#### Resolution:

1. Request a session token with MFA  
`aws sts get-session-token --serial-number arn-of-the-mfa-device --token-code code-from-token`. 

  `arn-of-the-mfa-device`: visible from your user IAM. 
  - Option: Use CLI to retrieve: `aws iam list-mfa-devices --user-name ryan`
  - Option: View in IAM console: IAM --> Users --> --> Security Credentials.  
  
   `code-from-token`: 6 digit code from your configured MFA device
  
2. Create a profile with the **returned** credentials.  
```
aws configure --profile sfl 
aws configure set --profile sfl aws_session_token <SESSION_TOKEN_HERE>
```
aws_session_token is not included in aws configure

3. Test command

`aws s3 ls --profile sfl`

4. Connect using boto3

```
session = boto3.Session(profile_name='sfl')
s3 = session.client('s3')
s3.upload_file(Filename,Bucket,Key)
```
### Building a layer from SAM

1. Creat a resoure in sam template like below:
```
Resources:
  WebhoseLayer:
    Type: AWS::Serverless::LayerVersion
    Properties:
      ContentUri: packages/
      CompatibleRuntimes:
        - python3.8
    Metadata:
      BuildMethod: python3.8
 ```
2. When create sam template, don't use !ref bucket twice, it will occur concurrency issues.
3. Create a requirement.txt file and put it into folder(packages)
4. run`sam build`, then the dependencies will be copy to the build file
5. `sam deploy --guided --profile [name]` (**don't delete samconfig.toml file**) 
