# Notes
## AWS 
### Issue 

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
aws configure set --profile cli aws_session_token <SESSION_TOKEN_HERE>
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

