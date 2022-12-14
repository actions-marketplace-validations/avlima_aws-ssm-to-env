# aws-ssm-to-env (GitHub Actions)

> Get AWS SSM Parameter values to environment variables.

## Table of Contents

  * [About](#about)
  * [Requirements](#requirements)
  * [Usage](#usage)
     * [Parameters](#parameters)
     * [Examples](#examples)
        * [Get single parameter value](#get-single-parameter-value)
        * [Get multiple parameter values](#get-multiple-parameter-values)
        * [Custom prefix](#custom-prefix)
        * [Simple JSON parameter values](#simple-json-parameter-values)
        * [Complex JSON values](#complex-json-values)
  * [TODO](#todo)

## About

This action is designed to read [AWS SSM parameters](https://console.aws.amazon.com/systems-manager/parameters) and 
exports them as environmental variables.

Script can parse string value parameters as well as parameters with stringified JSON values. For simple JSON objects
a shortcut parameter `simple_json` can be used to convert all key-values from JSON into environmental variables.

## Requirements

Action expects 3 secrets to be set in GitHub's repository:
- `AWS_REGION` - AWS Region (e.g. `us-east1`)
- `AWS_ACCESS_KEY_ID` - AWS Access Key for user with an SSM access policy (e.g. [AmazonSSMReadOnlyAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AmazonSSMReadOnlyAccess$serviceLevelSummary))
- `AWS_SECRET_ACCESS_KEY` - User's AWS Access Key

## Usage

### Parameters

Parameter name | Type | Required | Default Value | Description
--- | --- | --- | --- | ---
`ssm_parameter_list` | string | true | | AWS Systems Manager parameter name (path) or comma separated list of paths
`prefix` | string | false | AWS_SSM_ | Custom environmental variables prefix
`ssm_start_prefix` | string | false | | Filter SSM environment variables that starts with prefix.
`simple_json` | boolean | true | false | Parse parameter values as one-level JSON object and convert keys to environmental variables (see example below).
`jq_params` | string | true | | Custom space-separated [`jq` filters](https://stedolan.github.io/jq/) (see example below).

### Examples

#### Get single parameter value

Parse simple string value stored in AWS SSM `my_parameter_name` parameter:
```yaml
name: Parse SSM parameter

on:
  push

jobs:
  aws-ssm-to-env:
    runs-on: ubuntu-latest
    steps:
      - name: aws-ssm-to-env
        uses: avlima/aws-ssm-to-env@master
        env:
          AWS_REGION: ${{ secrets.AWS_REGION }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        with:
          ssm_parameter_list: 'my_parameter_name'
```

Example above will set environmental variable `AWS_SSM_MY_PARAMETER_NAME` with value from the AWS SSM parameter itself.

#### Get multiple parameter values

Use comma separated list of strings to fetch multiple parameter values at once:
```yaml
name: Parse SSM parameter

on:
  push

jobs:
  aws-ssm-to-env:
    runs-on: ubuntu-latest
    steps:
      - name: aws-ssm-to-env
        uses: avlima/aws-ssm-to-env@master
        env:
          AWS_REGION: ${{ secrets.AWS_REGION }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        with:
          ssm_parameter_list: |
           my_first_parameter,
           my_second_parameter
```

Example above will set environmental variable `AWS_SSM_MY_FIRST_PARAMETER` and `AWS_SSM_MY_SECOND_PARAMETER` with corresponding values from AWS SSM.

#### Custom prefix

Parse simple string value stored in AWS SSM `my_parameter_name` parameter and export environmental variable with a
custom prefix:
```yaml
name: Parse SSM parameter

on:
  push

jobs:
  aws-ssm-to-env:
    runs-on: ubuntu-latest
    steps:
      - name: aws-ssm-to-env
        uses: avlima/aws-ssm-to-env@master
        env:
          AWS_REGION: ${{ secrets.AWS_REGION }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        with:
          ssm_parameter_list: 'my_parameter_name'
          prefix: FOO_
```

Example above will set environmental variable `FOO_MY_PARAMETER_NAME` with value from the AWS SSM parameter itself.

#### SSM start prefix

Parse simple string value stored in AWS SSM `my_parameter_name` parameter and export environmental variable with a
custom prefix:
```yaml
name: Parse SSM parameter

on:
  push

jobs:
  aws-ssm-to-env:
    runs-on: ubuntu-latest
    steps:
      - name: aws-ssm-to-env
        uses: avlima/aws-ssm-to-env@master
        env:
          AWS_REGION: ${{ secrets.AWS_REGION }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        with:
          ssm_parameter_list: 'my_parameter_name'
          ssm_start_prefix: MY_PROJECT_
```

The above example will set the SSM environmental variable `MY_PROJECT_PARAMETER_NAME` to the local environmental variable `PARAMETER_NAME`, removing `MY_PROJECT_` from the beginning

#### Simple JSON parameter values

Parse simple one-level JSON object and create environmental variables from all keys:
```yaml
name: Parse JSON SSM parameter

on:
  push

jobs:
  aws-ssm-to-env:
    runs-on: ubuntu-latest
    steps:
      - name: aws-ssm-to-env
        uses: avlima/aws-ssm-to-env@master
        env:
          AWS_REGION: ${{ secrets.AWS_REGION }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        with:
          ssm_parameter_list: 'my_json_parameter'
          simple_json: true
```

If `my_json_parameter` in the example above is a JSON string like
```json
{"foo": "bar", "baz": 1}
```
environmental variables will be set as:
```sh
AWS_SSM_FOO=bar
AWS_SSM_BAZ=1
```

#### Complex JSON values

Pass a custom, space-separated filter(s) to `jq` and parse desired parts of JSON object:
```yaml
name: Parse JSON SSM parameter

on:
  push

jobs:
  aws-ssm-to-env:
    runs-on: ubuntu-latest
    steps:
      - name: aws-ssm-to-env
        uses: avlima/aws-ssm-to-env@master
        env:
          AWS_REGION: ${{ secrets.AWS_REGION }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        with:
          ssm_parameter_list: 'my_json_parameter'
          ssm_parameter_list: 'my_json_parameter'
          jq_filter: '.db[]|select(.default).host .db[]|select(.default).port'
          prefix: DB_
```

If `my_json_parameter` in the example above was a JSON string like
```json
{"db": [{"host": "my.db.host.com", "port": 1337, "default": true}, {"host": "other.host", "port": 42}]}
```
environmental variables will be set as:
```sh
DB_HOST=my.db.host.com
DB_PORT=1337
```

## TODO

 - [x] ~~Use official Docker container once it becomes available (https://github.com/aws/aws-cli/issues/3291, https://github.com/aws/aws-cli/issues/4685)~~
 - [ ] Write tests (https://github.com/kward/shunit2)
