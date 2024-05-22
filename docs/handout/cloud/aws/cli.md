

## Setting up the AWS Cli

![](../../../assets/images/cloud.aws.cli.crendentials.png)



``` shell
aws configure
```

<!-- termynal -->

``` shell
> aws configure
AWS Access Key ID: ****************5DMG
AWS Secret Access Key]: *********************************fhwQt
Default region name [None]: 
Default output format [None]: 
```


``` shell
aws sts get-session-token
```

<!-- termynal -->

``` shell
> aws sts get-session-token
{
    "Credentials": {
        "AccessKeyId": "ASIA4MTWJ5HP4RFKVFX2",
        "SecretAccessKey": "RWfqFn9NZRZYEy1a5sFpdUPSd5i03YRer/9+PZ6V",
        "SessionToken": "FwoGZXIvYXdzEJX//////////wEaDIRJrTOKnJTZ/
        ZpZGiKCAYnnc+16sxQl/eGYvj998q9u2eFb3VziCgpvNzKAuI/YcthL2XLp2VUXZswaOb5C3BikDENEKVbeH4va32ltJ/1Bm+F/
        qkHNE9dTRMOxshV9iwkCe3/4+Sl9O6dZJguglcCq2Yfh+9HDzJxo6WtAd7UiCL6C/
        hlcWgRS24IhvbdUDsgoy47qsQYyKNwLwW9ki4w5bmYRM9MVMinufs4LEkVRJGpEmc8
        RG3gNaGvnRB0d840=",
        "Expiration": "2024-05-08T07:55:55+00:00"
    }
}
```



``` shell
aws sts get-caller-identity
```

<!-- termynal -->

``` shell
> aws sts get-caller-identity
{
    "UserId": "AIDA4MTWJ5HPRUU7R22VG",
    "Account": "851725380063",
    "Arn": "arn:aws:iam::851725380063:user/root"
}
```





## Reference

* AWS Command Line Interface Documentation - [https://docs.aws.amazon.com/cli/](https://docs.aws.amazon.com/cli/){target='_blank'}
    
    - [User Guide - Install AWS Cli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html){target='_blank'}

        ![](../../../assets/images/cloud.aws.cli.install.png)

