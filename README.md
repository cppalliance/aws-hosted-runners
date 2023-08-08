
# AWS Hosted Runners

`aws-hosted-runners` is a GitHub Action that provides an output `labelmatrix` which can be used to configure the `runs-on` information in a workflow.

If self-hosted runners have been enabled, it will set the labels to `[ self-hosted, linux, x64, ubuntu-22.04-aws ]` or a similar value.  

If self-hosted runners are disabled, the existing label such as `ubuntu-22.04` will be kept.  

## Usage

These modifications should be made in the workflow file:  

- In the matrix section, quote the operating system labels. That is 'ubuntu-latest' or 'ubuntu-22.04'. Don't leave them as plain text ubuntu-latest. (The need for this change may depend on whether you have sections of json in the workflow or it is completely yaml format. Yaml may not require any modification).

- Add a `runner-selection` job:

```
jobs:
  runner-selection:
    runs-on: ubuntu-latest
    outputs:
      labelmatrix: ${{ steps.aws_hosted_runners.outputs.labelmatrix }}
    steps:
      - name: AWS Hosted Runners
        id: aws_hosted_runners
        uses: cppalliance/aws-hosted-runners@v1.0.0
        # with:
        #   self_hosted_runners_override: 'true'
        #   self_hosted_runners_url: 'https://example.com/switch'
        #   owner_list: example.com
        #   debug: 'true'
```

- Set the `runs-on` configuration in all jobs:

```
needs: [runner-selection]
runs-on: ${{ fromJSON(needs.runner-selection.outputs.labelmatrix)[matrix.os] }}
```

## Options

Usually not required.  

| Name          | Required | Default | Description                              |
| ------------- | -------- | ------- | ---------------------------------------- |
| self_hosted_runners_override | false  | (unset). The default is to refer to the _url setting. | If 'true', always use self-hosted runners. If 'false', never use self-hosted runners. |
| self_hosted_runners_url | false | https://gha.cpp.al/switch | Webpage which returns the directive if self-hosted runners ought to be used. |
| owner_list | false | boostorg | Space separated list of repository owners that will use self-hosted runners. | 
| debug | false | 'false' | Enable debugging |


## Fork Instructions

The original implementation of this Action was designed for a specific organization. All others are welcome to fork or clone the aws-hosted-runners repository and set up their own infrastructure.  
- In action.yml change the default value of `self_hosted_runners_url` so it points to your website instead.  
- In action.yml change the default value of `owner_list` so that it only contains your organization.  
- When modifying workflow files specify your information in the steps: `uses: _your_organization_/aws-hosted-runners@v1.0.0`  
- Configure Terraform to label self-hosted runners as `[ self-hosted, linux, x64, ubuntu-latest-aws ]` which has no label overlap with `ubuntu-latest`.  
- It is convenient to install a central admin server: https://github.com/cppalliance/github-runner-admin  

