# Using Telegram Bot with AWS services

## Main technologies and Programming Language

1. AWS Gateway
2. AWS Lambda
3. Telegram
4. Python 3

## 1. Get Telegram bot API Token

Create a new Telegram bot to get an API Token to interact with the platform. This will be required to establish the backend connection between AWS and Telegram bot. You can refer to the [official Telegram documentation](https://core.telegram.org/bots) to create a bot in section 3 - _How do I create a bot?_.

## 2. Create an AWS account

After you have signed up for an AWS account, you will be able to use services from AWS Free Tier for 12 months. However, if you 1) exceed the usage limit, 2) use a non Free Tier service or 3) no longer eligible for Free Tier, standard billing rates will be charged for your AWS account.

## 3. Create AWS Gateway

The AWS Gateway needs to be created to get the API endpoint. The API endpoint will look something like this:

`https://u3ir5tjcsf.execute-api.us-east-1.amazonaws.com/dev/my-custom-url`

where my-custom-url is the name you set for the API.

## 3.1 Set up webhook

You can set your webhook to allow your Telegram bot to receive calls from AWS when new data are received by entering the following link with the relevant parameters into your browser.

`https://api.telegram.org/bot<TELEGRAM_BOT_API_TOKEN>/setWebHook?url=<AWS_API_ENDPOINT>`

If the webhook is set successfully, you will receive a response:

```json
{
    "ok": true,
    "result": true,
    "description": "Webhook is already set"
}
```

## 3.2 Getting the chatbot id

You can get details of your chatbot id by calling your API endpoint. Below is a sample of how the response body look in JSON

```json
{
    "message":
    {
        "message_id": 39,
        "from": {
            "id": 183445107,
            "is_bot": false,
            "first_name": "Bernie",
            "last_name": "Choy",
            "username": "berniec",
            "language_code": "en"
        },
        "chat": {
            "id": 183445107,
            "first_name": "Bernie",
            "last_name": "Choy",
            "username": "berniec",
            "type": "private"
        },
        "date": 1582464438,
        "text": "Hi"
    }
}
```

## 4. Create AWS Lambda Function

The code snippet below will repeat what the user has sent by taking the input from the user's message and send it back to the user as a reply.

```python

import json
import requests

TELE_TOKEN='<TELEGRAM_BOT_API_TOKEN>'
URL = "https://api.telegram.org/bot{}/".format(TELE_TOKEN)

def send_message(text, chat_id):
    final_text = "You said: " + text
    url = URL + "sendMessage?text={}&chat_id={}".format(final_text, chat_id)
    requests.get(url)


def lambda_handler(event, context):

    json_obj = json.dumps(event)
    json_obj = json.loads(json_obj)
    body = json_obj['body']
    body = json.loads(body)
    message = body['message']
    chat = message['chat']
    id = chat['id']

    reply = message['text']
    send_message(reply, id)
    return {
        'statusCode': 200
    }

```

### 4.1 Deployment Package

In the event where you need to include other packages other than requests, you will have to create a deployment package i.e a ZIP archive that contains your function code and dependenccies.

> WARNING: When uploading the ZIP file, it will overwrite the entirety of the function. **Please ensure that the ZIP file consists of both your function code and the dependencies before uploading**.

```commandline

// Create a directory to contain the function codes & dependencies
mkdir zip_to_upload
cd zip_to_upload

// Install the relevant packages
pip3 install requests -t  zip_to_upload/
zip -r zip_to_upload.zip zip_to_upload/

// Install awscli via PyPi
pip3 install aws

// Use the update-function-code to upload the package via command line
aws lambda update-function-code --function-name <function_name> --zip-file fileb://<ZIP_filename>


// Alternatively, you can upload via the GUI interface via the Code entry type
// Select dropdown menu of Code entry type > Upload a .zip file

```

### 4.2 AWS Lambda with Deployment Package

Alternatively, you can upload the provided .zip file (sample-bot.zip) into your AWS Lambda where the pacakges required for requests has been installed.

## 5. Sources

1) Andrii Dvoiak,  [Serverless Telegram bot on AWS Lambda](https://medium.com/hackernoon/serverless-telegram-bot-on-aws-lambda-851204d4236c)

2) James Saryerwinnie, [Removing the vendored version of requests from Botocore](https://aws.amazon.com/blogs/developer/removing-the-vendored-version-of-requests-from-botocore/)

3) TheNiqabiCoderMum, [Building a Telegram Bot with AWS API Gateway and AWS Lambda](https://dev.to/nqcm/-building-a-telegram-bot-with-aws-api-gateway-and-aws-lambda-27fg)

4) AWS Official Documentation, [AWS Lambda Deployment Package](https://docs.aws.amazon.com/lambda/latest/dg/nodejs-package.html)

5) ragedsparrow, [Telegram Alert Action: Where do you get a "chat id"?](https://answers.splunk.com/answers/590658/telegram-alert-action-where-do-you-get-a-chat-id.html)
