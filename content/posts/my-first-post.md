---
title: "Who's on Support?"
date: 2021-04-24T23:56:13+01:00
draft: false
toc: false
images:
tags:
  - Posts
---

### Aim

Build a script that fetches data about who's on support each week and post it to Slack every Monday morning

## Tech Stack

- Python
- Slack incoming webhooks
- google spreadsheet api
- Amazon AWS Lambda

### Tutorial level:

intermediate

## Little background story

Currently at work, we have a weekly support rota system where we rotate support amongst each member of the team.
This information is stored in some Google sheets on the cloud and in order to find out who's on support each week you will have to manually check the sheet.

So I thought it would be nice to build a script that posts a message to my team's Slack channel with information about who's on support every Monday morning and to do that I did these steps

1. Follow a googlsheet tutorial to fetch data
2. Get an incoming webhook to my team's slack channel
3. Create an AWS lambda function to host my script
4. Write the script

## Boilerplate

### 1- Get the Spreadsheet

First I checked out the spreadsheet where we keep the support rota data and it currently looks a little like this

We have a primary person on support then a secondary person and then the manager. This is to make sure that there's always backup if one person doesnt respond in time

{{< image src="/images/support-rota-mock.png" alt="Hello Friend" position="center" style="border-radius: 8px; width: 400px" >}}

### 2- Slack incoming webhooks

To be able to post messages to Slack you need to get an incoming webhook and there are two ways to do that

1. To install a Slack app on your Slack workspace
2. To simply install an incoming webhook to your Slack channel

Initially while developing the script I created and installed a Slack app on my persoanl Slack workspace and once the script was ready I installed a custom incoming webhook (no app required) on my company's workspace and gave it acces to my team's channel so I can post messages there.

In either cases your incoming webhook should look like this
`https://hooks.slack.com/services/•••••••••••••/•••••••••••••••••••`

### 3- Google Dev API

Google has a simple and good tutorial on how to retreive data from a Google spreadsheet so I kept that open in a tab and chose the Python tutorial
`https://developers.google.com/sheets/api/quickstart/python`

### 4- Install Python3

For this one I used 3.7.3

### 4- Amazon AWS

You need an AWS account to run the script on a remote server

## Let's get started!

### GoogleSheets API Tutorial

1. Follow this google tutorial to create a simple script that loads data from Googlesheets`https://developers.google.com/sheets/api/quickstart/python`.
   _Gotcha_:
   Please be aware that `SAMPLE_RANGE_NAME` is the name of the sheet in your google sheets. Every googlesheet has multiple sheets and the value of this variable should be the name of the sheet you're getting the data from. My sheet was called `New` as in the screenshot

{{< image src="/images/support-rota-new-sheet-name.png" alt="Hello Friend" position="center" style="border-radius: 8px; width: 400px" >}}

2. Once your google script is hooked with your target Googlesheets we need to do the following

- You will notice that in the Google tutorial you will have generated a `credentials.json` and `tokens.json` files. We are going to move the content of the two files into a local object in the main file where the script is so that when we host the script on AWS we wont get into issues with opening/writing to a file.

Move the content of each file into a separate object in your script file like so

```
credentialsObject = {
    "installed": {
    "client_id": "******************************************************",
    "project_id": "******************************************************",
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
    "client_secret": "******************************************************",
    "redirect_uris": ["urn:ietf:wg:oauth:2.0:oob", "http://localhost"]
    }
}
tokensObject = {
    "token": "************************************************************************************************************",
    "refresh_token": "************************************************************************************************************",
    "token_uri": "https://oauth2.googleapis.com/token",
    "client_id": "******************************************************",
    "client_secret": "*************************",
    "scopes": ["https://www.googleapis.com/auth/spreadsheets.readonly"],
    "expiry": "2021-04-12T09:43:06.121345Z"
}

```

3. Next we will use our local objects instead of importing the json files

- Credentials: Where it says

```
flow = InstalledAppFlow.from_client_secrets_file('credentials.json', SCOPES)
```

we are going to replace that with the line below so that we can use our `credentialsObject` instead of using the credentials.json file

```
InstalledAppFlow.from_client_secrets_file(credentialsObject, SCOPES)
```

- Tokens: Where it says

```
creds = Credentials.from_authorized_user_file('token.json', SCOPES)
```

we will change that to the line below so that we use our tokensObject instead of using the tokens.json file

```
Credentials.from_authorized_user_info(tokensObject, SCOPES)
```

### Collecting Information for the script

When posting the support rota message, I wanted to tag the person on support on Slack and to do that I created a map with each team member's name and their slack ID.

In order to get the slack ID you need to click on the person's avatar on Slack > `More` > `Copy Member ID` like in the screen shot below

{{< image src="/images/slackID.png" alt="Hello Friend" position="center" style="border-radius: 8px; width: 400px" >}}

and then the map will look like this

```
rotaNametoSlackID = {
        "Wafa Zaidan": { slackID: "********"},
}
```

### Writing Our Script

1. Right after `values = result.get('values', [])` I will just return values so that the `main` function's responsibility would be to only fetch the information from the sheets and return them

So the function will look like this now

```
   def main():
        creds = None
        creds = Credentials.from_authorized_user_info(tokensObject, SCOPES)
        if not creds or not creds.valid:
            if creds and creds.expired and creds.refresh_token:
                creds.refresh(Request())
            else:
                flow = InstalledAppFlow.from_client_secrets_file(
                    credentialsObject, SCOPES)
                creds = flow.run_local_server(port=0)
        service = build('sheets', 'v4', credentials=creds)

        result = service.spreadsheets().values().get(
        spreadsheetId=SAMPLE_SPREADSHEET_ID, range=SAMPLE_RANGE_NAME).execute()
        values = result.get('values', [])
        return values
```

2. Next we will create a new function called `getNamesOnSuport` which accepts the values that are returned from the `main` function as a parameter. The function then will do some magic to return the slack ID of the person on support that week

Now I want this script to fire every Monday morning as thats the beginning of the working week and the date stored in my team's Googlesheets.
First I will import Python's `datetime` library. After that in order to get the date today I will call `datetime.datetime.today()` then call `strftime()` on it so that it returns the date in a format similar to the date format stored in our googlesheet so that we are able to compare them

```
datetime.date.strftime(datetime.datetime.today(), "%d/%m/%Y")
```

Then we will loop through the values we got from the `main` function and compare each date to today's formatted date and once we find a match we will will get the name of the person in that row and then we will loop through our `rotaNametoSlackID` map to return the slack id for that name

So the function should look like this

```
def getNamesOnSuport(values):
    formattedDateToday = datetime.date.strftime(datetime.datetime.today(), "%d/%m/%Y")
            peopleOnSuport = ""
            for i in values:
                if i[0] == formattedDateToday:
                    for name in rotaNametoSlackID:
                        personOnSupportSlackID = rotaNametoSlackID[name]["slackID"]
                        peopleOnSuport+= f"<@{personOnSupportSlackID}>"
                        return peopleOnSuport

```

3. After that I created function `postMessageToSlack` which will post a message with the information I have to my team's Slack channel.

   - First I imported Python's http library `import http.client`

   - Then I stored the slack token I generted from the incoming webhooks in a local variable called `PERSONAL_SLACK_TOKEN`

   - Next I created a https connection, prepared the headers and the data and finally added the code to actually make the request

And the function looked like this

```
def postMessageToSlack(message):
    slackConnection = http.client.HTTPSConnection("hooks.slack.com")
    headers = {"Content-Type": "application/json"}
    data = "{'text':'"+message+"'}"
    slackConnection.request(
        "POST",
        f"/services/{PERSONAL_SLACK_TOKEN}",
        data,
        headers
    )
```

4. Now its time to call these functions.

   - First I called `main` and stored the values it returns in a variable
     `sheetData = main()`
   - Then I called `getNamesOnSuport(sheetData)` and passed it the values we got from `main` and stored the returned value in a local variable too
     `peopleOnSuport = getNamesOnSuport(sheetData)`
   - Next I created the message that I want to post to my team's Slack channel by adding a simple ternary operation that returns a specific message if there are peopel on support and a different one if there are none

     `rotaMessage = f"On the support rota this week {peopleOnSuport} :superhero: Good luck! :fire: :bomb: " if peopleOnSuport else 'No one is on support this week!'`

- Finally I simply called `postMessageToSlack` with the message we created to post it to Slack
  `postMessageToSlack(rotaMessage)`

At this point I ran the script by executing `python3 quickStart.py` and I saw a message popping up in my target Slack channel like this

{{< image src="/images/message-slack.png" alt="Hello Friend" position="center" style="border-radius: 8px; width: 400px" >}}

### Hosting the script

In order to automate the firing of this script so that we dont have to manually call it every time, we need to run it on a server.

You could use a raspberry pi if you're ambitious but I went for a lambda function on AWS and because its a simple script, a lambda should do

After logging into my AWS account, I went to `Services` > `lambda` > `Create function` > Gave it a function name and chose python3.8 as the runtime language > `Create function`

You will see a page that looks like this and a file called `lambda_function.py` already created there. Double click on it to open and you will notice a function called `lambda_handler`, this is a key function that AWS looks for when running our script.
Within the function you will notice a commented out line which indicates where our code should go

{{< image src="/images/aws-lambda.png" alt="Hello Friend" position="center" style="border-radius: 8px; width: 400px" >}}

Next I wrapped my script in the `lambda_handler` key function to make it AWS ready

1. We will change the name of our script's file from `quickStart.py` to `lambda_function.py` as its the file name that AWS will look for when running the code in the server
2. Next I added this line at the top of my script to import Pythons's JSON library `import json`
3. then I wrapped the script in a new function called `lambda_handler(event, context)` and made it accept two parameters - This is needed to mimic the structure of the lambda function that AWS is expecting

4. Right at the end of the script I added this return statement

```
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
```

5. Next I followed this AWS tutorial `https://docs.aws.amazon.com/lambda/latest/dg/python-package-create.html#python-package-create-with-dependency` to zip the lambda function with the google dependencies we have.

   Usually For a basic script with no external runtime dependencies I would have just copied and pasted the code as it is at this point in the `lambda_function.py` but because we have some runtime dependencies we have to zip a folder with the dependencies and the script.

- Be careful when running this command which is mentioned in the tutorial

```
  pip install --target ./package requests
```

as you need to install the Google dependencies and not the Python `request` library.
So the command should look like this instead

```
pip install --target ./package --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib
```

- And these are the exact steps I did

```
mkdir my-sourcecode-function
Moved the `lambda_function.py` file I created into the new my-sourcecode-function`
cd my-sourcecode-function
pip install --target ./package --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib
cd package
zip -r ../my-deployment-package.zip .
cd ..
zip -g my-deployment-package.zip lambda_function.py
```

And this will ship my script and the dependcies ina new zip folder which will appear in the root folder

6. Next its time to upload the zip folder.

   I went back to the AWS function page and clicked on `Upload Form` > `.zip file` and uploaded the zip folder I created and clicked `Save`

{{< image src="/images/aws-zip.png" alt="Hello Friend" position="center" style="border-radius: 8px; width: 400px" >}}

7. Next I configured a time limit for this function to run so it doesnt exceed a certain timeout
   I went to `Configuration` > `General Confuguration` > `Edit` > Set the timeout to 1 minute

{{< image src="/images/aws-time.png" alt="Hello Friend" position="center" style="border-radius: 8px; width: 400px" >}}

- I would also recommend setting a budget on your AWS account so it doesnt exceed a certain amount so that you dont end up with a surprise bill. I set my amount to $2

8. After that I clicked the `Test` tab then the `Test` button to see if the function runs correctly and posts a message to Slack and it did in my case yayyyy!

9. Next I set a cron job to make the function run automatically at a certain day/time. So back to the AWS function and clicked on the `Add trigger` button

{{< image src="/images/trigger.png" alt="Hello Friend" position="center" style="border-radius: 8px; width: 400px" >}}

Then I chose `EventBridge (CloudWatch Events)` and added a cron job.
Mine looked like this `cron(32 8 ? * MON-MON *)` as I want the script to run every Monday at 8:32

Et Voilaaa!!! The script is ready and is in the cloud, now all I have to do is set back and relax and expect it to fire every Monday morning

You can find out how the final script that I built looks like here https://github.com/WafaZaidan/who-is-on-support
