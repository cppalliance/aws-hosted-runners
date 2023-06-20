
# AWS Hosted Runners

`aws-hosted-runners` is a GitHub Action that provides an output `labelmatrix` which can be used to configure the `runs-on` information in a workflow.

If self-hosted runners have been enabled, it will set the labels to `[ self-hosted, linux, x64, ubuntu-22.04-aws ]` or a similar value.  

If self-hosted runners are disabled, the existing label such as `ubuntu-22.04` will be kept.  

## Usage

These modifications should be made in the workflow file:  

- In the matrix section, quote the operating system labels. That is 'ubuntu-latest' or 'ubuntu-22.04'. Don't leave them as plain text ubuntu-latest.

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


## Instructions

See https://github.com/cppalliance/githubactions for instructions.  

This implementation is primarily aimed at boost.org and cppalliance.org users. However others are welcome to clone the repository and set up their own infrastructure. The main difference if using this elsewhere is to change the default value of `self_hosted_runners_url` so it points to your website instead.  
