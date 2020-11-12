#Scheduling a bot reminder with Cloud Functions and Cloud Scheduler

We're assuming that you are [introduced to webhooks](https://developers.google.com/hangouts/chat/how-tos/webhooks), and you can obtain a `CHAT_WEBHOOK_URL` from Google Chat.

First you can do a test bot with a new chat room (and a new webhook):
```
import requests
import json
# Go to the chat room, click manage webhook and create a webhook
# Copy the link and paste it here.

URL = 'CHAT_WEBHOOK_URL'
message = {'text': 'Hello,\n This is a test message.'}

requests.post(URL,
        data = json.dumps(message))
```

If you are using a GCE VM instance, you may first have to update the scopes of the service account you'll use for running the function.
```
 gcloud compute instances set-service-account VM_NAME --service-account SERVICE_ACCOUNT --scopes cloud-platform
```

It's time to introduce our previous test in a cloud function. Our function will use the script from the file `main.py`. You'll need to set your `CHAT_WEBHOOK_URL` and the message you want to post as `CHAT_MESSAGE`.
```
"""Function called by PubSub trigger to execute cron job tasks."""
import requests
import json
import datetime
import logging
from string import Template

def main(data,context):
# Go to the chat room, click manage webhook and create a webhook
# Copy the link and paste it here.
    try:
        current_time = datetime.datetime.utcnow()
        log_message = Template('Cloud Function was triggered on $time')
        logging.info(log_message.safe_substitute(time=current_time))
   
   
        try:
            URL = 'CHAT_WEBHOOK_URL'
           
            message = {'text': CHAT_MESSAGE}
           
            requests.post(URL,
                    data = json.dumps(message))
        except Exception as error:
            log_message = Template('Query failed due to '
                                   '$message.')
            logging.error(log_message.safe_substitute(message=error))

    except Exception as error:
        log_message = Template('$error').substitute(error=error)
        logging.error(log_message)

if __name__ == '__main__':
    main('data','context')
```

In the same directory as `main.py` has been saved, we must create the function, as well as the PubSub topic that will orchestrate the scheduling.
```
gcloud functions deploy chat_function --entry-point main --runtime python37 --trigger-resource chat-topic --trigger-event google.pubsub.topic.publish --timeout 540s

```

Now it's time to set the scheduled job:
```
gcloud scheduler jobs create pubsub chat_job --schedule "00 12 * * 1,2,3,4,5" --time-zone "Etc/GMT-1"  --topic chat-topic --message-body "This runs at 12!"
```

## Useful links:

* [How to schedule a recurring Python script on GCP](https://cloud.google.com/blog/products/application-development/how-to-schedule-a-recurring-python-script-on-gcp)
* [Cron schedule expressions editor](https://crontab.guru/)
* [GCE VM and Service Accounts I](https://cloud.google.com/compute/docs/access/service-accounts#accesscopesiam)
* [GCE VM and Service Accounts II](https://cloud.google.com/compute/docs/access/create-enable-service-accounts-for-instances#gcloud_1)
* [Intro to webhooks](https://developers.google.com/hangouts/chat/how-tos/webhooks)
