# RFDK Sample Application - Deadline - Typescript

## Overview
[Back to overview](../README.md)

## Instructions

---
**NOTE**

These instructions assume that your working directory is `examples/deadline/All-In-AWS-Infrastructure-Basic/ts/` relative to the root of the RFDK package.

---

1. This sample app on the `mainline` branch may contain features that have not yet been officially released, and may not be available in the `aws-rfdk` package installed through npm from npmjs. To work from an example of the latest release, please switch to the `release` branch. If you would like to try out unreleased features, you can stay on `mainline` and follow the instructions for building and using the `aws-rfdk` from your local repository.
2.  Install the dependencies of the sample app:

    ```
    yarn install
    ```
3.  By downloading or using the Deadline software, you agree to the [AWS Customer Agreement](https://aws.amazon.com/agreement/)
    and [AWS Intellectual Property License](https://aws.amazon.com/legal/aws-ip-license-terms/). You acknowledge that Deadline
    is AWS Content as defined in those Agreements.
    To accept these terms, change the value of `acceptAwsCustomerAgreementAndIpLicense` in `bin/config.ts`:

    ```ts
    public readonly acceptAwsCustomerAgreementAndIpLicense: AwsCustomerAgreementAndIpLicenseAcceptance = AwsCustomerAgreementAndIpLicenseAcceptance.USER_REJECTS_AWS_CUSTOMER_AGREEMENT_AND_IP_LICENSE;
    ```

4.  Change the value of the `deadlineVersion` variable in `bin/config.ts` to specify the desired version of Deadline to be deployed to your render farm. RFDK is compatible with Deadline versions 10.1.9.x and later. To see the available versions of Deadline, consult the [Deadline release notes](https://docs.thinkboxsoftware.com/products/deadline/10.1/1_User%20Manual/manual/release-notes.html). It is recommended to use the latest version of Deadline available when building your farm, but to pin this version when the farm is ready for production use. For example, to pin to the latest `10.1.12.x` release of Deadline, use:

    ```ts
    public readonly deadlineVersion: string = '10.1.12';
    ```

5.  Change the value of the `deadlineClientLinuxAmiMap` variable in `bin/config.ts` to include the region + AMI ID mapping of your EC2 AMI(s) with Deadline Worker. You can use the following AWS CLI query to find AMI ID's:
    ```
    aws --region <region> ec2 describe-images \
    --owners 357466774442 \
    --filters "Name=name,Values=*Worker*" "Name=name,Values=*<version>*" \
    --query 'Images[*].[ImageId, Name]' \
    --output text
    ```

    And enter it into this section of `bin/config.ts`:
    ```ts
    // For example, in the us-west-2 region
    public readonly deadlineClientLinuxAmiMap: Record<string, string> = {
      ['us-west-2']: '<your-ami-id>',
      // ...
      };
    ```

    ---

    **Note:** The next three steps are for setting up usage based licensing and are optional. You may skip these if you do not need to use licenses for rendering.

    ---
6.  Create a binary secret in [SecretsManager](https://aws.amazon.com/secrets-manager/) that contains your [Usage-Based Licensing](https://docs.thinkboxsoftware.com/products/deadline/10.1/1_User%20Manual/manual/aws-portal/licensing-setup.html?highlight=usage%20based%20licensing) certificates in a `.zip` file:

    ```
    aws secretsmanager create-secret --name <name> --secret-binary fileb://<path-to-zip-file>
    ```
7.  The output from the previous step will contain the secret's ARN. Change the value of the `ublCertificatesSecretArn` variable in `bin/config.ts` to the secret's ARN:

    ```ts
    public readonly ublCertificatesSecretArn: string = '<your-secret-arn>';
    ```
8.  Choose your UBL limits and change the value of the `ublLicenses` variable in `bin/config.ts` accordingly. For example:

    ```ts
    public readonly ublLicenses: UsageBasedLicense[] = [
      // your UBL limits, for example:

      // up to 10 concurrent Maya licenses used at once
      UsageBasedLicense.forMaya(10),

      // unlimited Arnold licenses
      UsageBasedLicense.forArnold()
    ];
    ```

    ---

    **Note:** The next two steps are for allowing SSH access to your render farm and are optional. You may skip these if you do not need SSH access into your render farm.

    ---
9.  Create an EC2 key pair to give you SSH access to the render farm:

    ```
    aws ec2 create-key-pair --key-name <key-name>
    ```
10. Change the value of the `keyPairName` variable in `bin/config.ts` to your value for `<key-name>` in the previous step:

    **Note:** Save the value of the `"KeyMaterial"` field as a file in a secure location. This is your private key that you can use to SSH into the render farm.

    ```ts
    public readonly keyPairName: string = '<key-name>';
    ```
11. Choose the type of database you would like to deploy (AWS DocumentDB or MongoDB).
    If you would like to use MongoDB, you will need to accept the Mongo SSPL (see next step).
    Once you've decided on a database type, change the value of the `deployMongoDB` variable in `bin/config.ts` accordingly:

    ```ts
    // true = MongoDB, false = Amazon DocumentDB
    public readonly deployMongoDB: boolean = false;
    ```
12. If you set `deployMongoDB` to `true`, then you must accept the [SSPL license](https://www.mongodb.com/licensing/server-side-public-license) to successfully deploy MongoDB. To do so, change the value of `acceptSsplLicense` in `bin/config.ts`:

    ```ts
    // To accept the MongoDB SSPL, change from USER_REJECTS_SSPL to USER_ACCEPTS_SSPL
    public readonly acceptSsplLicense: MongoDbSsplLicenseAcceptance = MongoDbSsplLicenseAcceptance.USER_REJECTS_SSPL;
    ```
13. Optionally configure alarm notifications. If you choose to configure alarms, change the value of the `alarmEmailAddress` variable in `bin/config.ts` to the desired email address to receive alarm notifications:

    ```ts
    public readonly alarmEmailAddress?: string = 'username@yourdomain.com';
    ```

15. Deadline Secrets Management is a feature used to encrypt certain values in the database that need to be kept secret. Additional documentation about the feature and how it works in the RFDK can be found in the [RFDK README](../../../../packages/aws-rfdk/lib/deadline/README.md). By default, Deadline Secrets Management is enabled, but it can be disabled by changing the `enableSecretsManagement` variable in `package/config.ts`.

    ```ts
    public readonly enableSecretsManagement: boolean = false;
    ```

16. When you are using Deadline Secrets Management you can define your own admin credentials by creating a Secret in AWS SecretsManager in the following format:

    ```json
        {
            "username": "<admin user name>",
            "password": "<admin user password>",
        }
    ```
    The password must be at least 8 characters long and contain at least one lowercase, one uppercase, one digit, and one special character.

    Then the value of the `secretsManagementSecretArn` variable in `package/config.ts` should be changed to this secret's ARN:

    ```ts
    public readonly secretsManagementSecretArn?: string = '<your-secret-arn>';
    ```
    It is highly recommended that you leave this parameter undefined to enable the automatic generation of a strong password.

14. Build the `aws-rfdk` package, and then build the sample app. There is some magic in the way yarn workspaces and lerna packages work that will link the built `aws-rfdk` from the base directory as the dependency to be used in the example's directory:
    ```bash
    # Navigate to the root directory of the RFDK repository (assumes you started in the example's directory)
    pushd ../../../..
    ./build.sh
    # Navigate back to the example directory
    popd
    # Run the example's build
    yarn build
    ```
15. Deploy all the stacks in the sample app:

    ```
    cdk deploy "*"
    ```
16. Once you are finished with the sample app, you can tear it down by running:

    ```
    cdk destroy "*"
    ```
