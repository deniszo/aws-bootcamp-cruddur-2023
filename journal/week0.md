# Week 0 — Billing and Architecture

[Class video](https://www.youtube.com/watch?v=SG8blanhAOg&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=13) | 

This week was dedicated to getting to know the tools such as AWS console, AWS CLI, Lucid Charts and Gitpod.

To kick off working with AWS, we first set up a separate IAM (**I**dentity and **A**ccess **M**anagement) account to use it with AWS CLI. Using separate IAM accounts is good practice in terms of security because it enables limiting which resources a particular account has access to (Billing, Compute, etc.) and defining what permissions over those resources that account has (Create, Read, Update, Delete). These settings are defined by **policies**, which can be very broad in terms of what they allow like `AdministratorAccess` enabling (you guessed it) admin-like access to almost all of the resources (although it doesn't seem to provide access to billing?), so naturally it is encouraged to use more narrow policies, but for the sake of simplicity we resorted to the former one.

Having created the IAM user and its Access Key (along with the secret key) we proceeded to configuring the Gitpod, which is our way to have a unified dev-environment and is similar to Github's codespaces but provides more generous free-tier. 

### Dev env setup

While Console is great for some quick adjustments its still manual work, so the next logical step was to see the ways to automate everything with AWS CLI. There are multiple ways to run it and normally one would simply use their machine for it. Alternatively, AWS recommends using *CloudShell* which is an in-Browser shell emulator with AWS CLI and other common tools installed and configured.

In our case we had to a bit of setup to get it running in the Gitpod environment, but it was quite easy and what's more important Gitpod supports configuration via `.gitpod.yml` file in which we added the following lines:

```yaml
tasks:
  - name: aws-cli
    env:
      AWS_CLI_AUTO_PROMPT: on-partial
    init: |
      cd /workspace
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
      cd $THEIA_WORKSPACE_ROOT
```

This task basically downloads the AWS CLI and installs it in on startup.

Another thing was to set up AWS env variables so that CLI is able to automatically authenticate the user. Here again Gitpod provides `gp env` command to persist those env variables between launches and we did just that:

```bash
gp env AWS_ACCESS_KEY_ID="access key value"
gp env AWS_SECRET_ACCESS_KEY="secret key value"
gp env AWS_DEFAULT_REGION="us-west-2"
```

To ensure the settings are picked up by AWS CLI the following command can be used:
```bash
$ aws sts get-caller-identity
{
    "UserId": "{redacted}",
    "Account": "{redacted}",
    "Arn": "arn:aws:iam::610340716940:user/aws-bootcamp-2023"
}
```

### Bugdets

We started our AWS journey with a quick Billing overview and to control spending (I am no longer elegilble for a free-tier so super important stuff for me) we first manually created a "Monthly Budget" via Console. Budgets are great ways to track spending starting from just the amount in selected currency up to granular quotas per service. Pricing is very involved topic on it's own and we got a very helpful [pricing basics video](https://www.youtube.com/watch?v=OVw3RrlP-sI&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=14) along with some extra [spedning considerations](https://docs.google.com/document/d/10Hec7Or1ZUedl0ye-05mVPhYFR5-ySh2K8ZbFqTxu1w/edit#) in writing.

Things to note:
- AWS billing estimation is based on **730** hours per months (which is more than 30 days or 720 hours but less than 31 days of 744 hours)

After initial overview we swtiched to CLI and set up a budget along with *CloudWatch* alert (example can be found [here](https://aws.amazon.com/premiumsupport/knowledge-center/cloudwatch-estimatedcharges-alarm/))

```bash
export $AWS_ACCOUNT_ID = $(aws sts get-caller-identity --query Account --output text)

aws budgets create-budget \
    --account-id $AWS_ACCOUNT_ID \
    --budget file://aws/json/budget.json \
    --notifications-with-subscribers file://aws/json/budget-notifications.json

# create SNS topic for alarm and use its ARN for subscription
export AWS_BUDGET_ALARM_TOPIC_ARN = $(aws sns create-topic --name billing-alarm --query TopicArn)

# subscription needs to be confirmed via link in email
aws sns subscribe \
    --topic-arn=$AWS_BUDGET_ALARM_TOPIC_ARN \
    --protocol=email \
    --notification-endpoint=dezolkin@gmail.com

aws cloudwatch put-metric-alarm --cli-input-json file://aws/json/alarm.json
```

[This video](https://www.youtube.com/watch?v=OdUnNuKylHg&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=15) basically does everything described above