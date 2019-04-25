# CloudFormation-Template-Development-CI-CD-Pipeline

## Summary
CloudFormation template to create CI/CD Pipeline used to continuously develop CloudFormation templates.

---

## Description

This template creates the following:

- CodeCommit Respository
- CodePipeline with the following:

    - CodeCommit Repo as the source stage
    - CloudFormation as the deploy stage

- S3 bucket for CodePipeline artifacts
- Custom Resource that does the following:
    - During stack creation:

        - Performs an initial commit to this repo with the following files:

            - `template.yml`: CloudFormation template file
            - `input-params.json`: File that holds the input parameter values to be passed to CloudFormation during create-stack and update-stack operations

                - Reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/continuous-delivery-codepipeline-cfn-artifacts.html#w2ab1c13c15c15

    - During stack deletion:
        - removes all objects from the S3 bucket so that it can successfully be deleted by the CloudFormation stack 

--- 

## Use

`aws cloudformation create-stack --stack-name CloudFormation-Template-Development-CI-CD-Pipeline --template-body file://CloudFormation-Template-Development-CI-CD-Pipeline.yml --capabilities CAPABILITY_NAMED_IAM`

---

## Purpose

Having a CI/CD Pipeline for CloudFormation template creation is extremely useful.  You can begin building your template in the `template.yml` file in the git repo, and as each resource is added (or a small group of resources) you can do the following:

- validate the template with the following command:

    `aws cloudformation validate-template --template-body file://template.yml`

- commit changes to the local git repo, and then push the changes to the remote repo with `git push`

Once the CodeCommit repo has a new commit, the Pipeline is triggered and will perform an update-stack operation on the CloudFormation stack for this template.

This way you can continually add to the template and know that the resources that you have created are able to successfully launch without issues.

---

## Known issues:

- Lambda function for Custom Resource is performing a CreateCommit API call to CodeCommit using the boto3 SDK. However, the boto3 package that is provided to lambda functions by default is so out of date that this method is unsupported.  I have created a Lambda layer in my account to provide the function with an updated version of boto3 
- Currently the [Klayers project](https://github.com/keithrozario/Klayers) on GitHub is working to add a public Lambda layer for boto3.  Once this is finished, I will update the template to reference that public Lambda layer.  
- See GitHub Issue: https://github.com/keithrozario/Klayers/issues/26