# cfn-macro-ssm-param

The `SsmParam` macro allows retrieval of parameters from the SSM Parameter Store.

Inventory of source code and supporting files:

- ssm_param - Code for the application's Lambda function.
- events - Invocation events that you can use to invoke the function.
- tests - Unit tests for the application code.
- template.yaml - A template that defines the application's AWS resources.

## Example
The following contains an example of how to use the transform and the resulting output.

### Before
This is a segment of a Cloudformation template inside a
`AWS::CloudFormation::Init` definition where a secret key is used using
`Fn::Transform`:

```yaml
commands:
  01_jumpcloud_agent:
    command: !Join
      - ''
      - - "/usr/bin/curl --silent --show-error --header 'x-connect-key: "
        - Fn::Transform
          Name: SsmParam
          Parameters:
            Type: SecureString
            Name: my-secret-key
        - "' https://kickstart.jumpcloud.com/Kickstart | sudo bash"
```

### After
This is what the template segment will look like after the macro runs:

```yaml
commands:
  01_jumpcloud_agent:
    command: !Join
      - ''
      - - "/usr/bin/curl --silent --show-error --header 'x-connect-key: "
        - "my-secret-key-value"
        - "' https://kickstart.jumpcloud.com/Kickstart | sudo bash"
```

## Deploy the sample application

The Serverless Application Model Command Line Interface (SAM CLI) is an extension of the AWS CLI that adds functionality for building and testing Lambda applications. It uses Docker to run your functions in an Amazon Linux environment that matches Lambda. It can also emulate your application's build environment and API.

## Use the SAM CLI to build and test locally

### Setup Development Environment

Install the following applications:
* [AWS CLI](https://github.com/aws/aws-cli)
* [AWS SAM CLI](https://github.com/aws/aws-sam-cli)
* [pre-commit](https://github.com/pre-commit/pre-commit)
* [pipenv](https://github.com/pypa/pipenv)

### Install Requirements

Run `pipenv install --dev` to install both production and development
requirements, and `pipenv shell` to activate the virtual environment. For more
information see the [pipenv docs](https://pipenv.pypa.io/en/latest/).

After activating the virtual environment, run `pre-commit install` to install
the [pre-commit](https://pre-commit.com/) git hook.

### Update Requirements

First, make any needed updates to the base requirements in `Pipfile`, then use
`pipenv` to regenerate both `Pipfile.lock` and `requirements.txt`.

```shell script
$ pipenv update --dev
```

We use `pipenv` to control versions in testing, but `sam` relies on
`requirements.txt` directly for building the lambda artifact, so we dynamically
generate `requirements.txt` from `Pipfile.lock` before building the artifact.
The file must be created in the `CodeUri` directory specified in
`template.yaml`.

```shell script
$ pipenv requirements > ssm_param/requirements.txt
```

Additionally, `pre-commit` manages its own requirements.

```shell script
$ pre-commit autoupdate
```

### Create a local build

Use a Lambda-like docker container to build the Lambda artifact

```shell script
$ sam build --use-container
```

### Run unit tests

Tests are defined in the `tests` folder in this project, and dependencies are
managed with `pipenv`. Install the development dependencies and run the tests
using `coverage`.

```shell script
$ pipenv run coverage run -m pytest tests/ -svv
```

Automated testing will upload coverage results to [Coveralls](coveralls.io).

### Run integration tests

Running integration tests
[requires docker](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-local-start-api.html)

```shell script
$ sam local invoke SsmParamFunction --event events/event.json
```

## Fetch, tail, and filter Lambda function logs

To simplify troubleshooting, SAM CLI has a command called `sam logs`. `sam logs` lets you fetch logs generated by your deployed Lambda function from the command line. In addition to printing the logs on the terminal, this command has several nifty features to help you quickly find the bug.

`NOTE`: This command works for all AWS Lambda functions; not just the ones you deploy using SAM.

```bash
$ sam logs -n SsmParamFunction --stack-name cfn-macro-ssm-param --tail
```

You can find more information and examples about filtering Lambda function logs in the [SAM CLI Documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-logging.html).

## Cleanup

To delete the sample application that you created, use the AWS CLI. Assuming you used your project name for the stack name, you can run the following:

```bash
aws cloudformation delete-stack --stack-name cfn-macro-ssm-param
```

## Author

[Tess Thyer](https://github.com/tthyer); Sr. Data Engineer, Sage Bionetworks
