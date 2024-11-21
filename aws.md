There is a profile assocated with each IAM user or role.
A profile has its own `Access Key ID`(think of it as username) and `Secret Access Key`(think of it as password).  
To use these 2 credentials you can put them in `~/.aws/credentials` like this:
```
[YourProfileName]
aws_access_key_id = AKIAEXAMPLEEXAM
aws_secret_access_key = Orwr8yWoQK7MDENG/bPxRfiCYEXAMPLEANOTHERKEY
```

There is also a default profile so that if you don't specify a profile, it will be used.

You can specify multiple profile in the same credentials file, like this:
```
[Profile1]
aws_access_key_id =
aws_secret_access_key = 

[Profile2]
aws_access_key_id =
aws_secret_access_key = 
```

The profile name can be chosen by you. AWS only identifies the key and key id not the profile name.
