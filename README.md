# Home Assistant Webhook Automation for Vine Helper

I've created this repository to help Home Assistant users who are also members of Amazon Vine programme utilise the Webhook feature of VineHelper to generate Home Assistant notificiations when new RFY items are found, AFA items listed, Keywords matched, or Zero ETV items found utilising the Discord Webhooks of Vine Helper. These notification can be sent to a mobile phone, smart speaker, or any other smart device connected to Home Assistant.

Notifications are near instant.

## What does it do?

This Webhook will send a notification from Home Assistant of a new item on Vine. The automation checks what category of item it is (eg. RFY, Keyword match) and formats the output accordingly.

The notification will have the title of the category, E.g. "New item in RFY!", the name of the item, the photo of the item, and the notification will link to the item.

On iOS, because an iOS device uses Universal Links to open the relevant app, the links will open in the Amazon App if installed, or the default browser if not. On Android it may open in the Amazon App or the default browser. I recommend having your browser logged into Amazon for ease of ordering.

Because it's difficult to get Vine Helper to run on a Mobile Device, and because the links may open in the Amazon App, the links will take you to your RFY page for RFY items, and for other items will use the Vine Search to search for the title of the item returned. In 99.9% of cases this will only show the item notified about, but may occasionally show other similar items too.

From here you can order the item in the normal way by clicking the "See details" link.

## Prerequesits

### Membership of Amazon Vine

You must already be a member of the Amazon Vine Community. Membership is by invite only and selected by Amazon.

### Vine Helper

This automation template uses the Discord Webhooks from the [Vine Helper browser extension](https://www.vinehelper.ovh/) and expects the standard JSON payload.

For automations to be triggered, you must have a browser open and running the Vine Helper extension and open on the Notifications Monitor screen.

### Home Assistant

You should already have a [Home Assistant](https://www.home-assistant.io/) (HA) server up and running. If you are new to HA see the "Getting started" guidance.

#### Cloud Access

You will also require access to your Home Assistant server from outside the home LAN. There are lots of ways of achieving this both paid and free, but I recommend [Nabu Casa](https://www.nabucasa.com/) which requires a paid subscription, but which supports the development of Home Assistant. My guidance uses Nabu Casa as the remote access method.

#### Home Assistant Companion App

To send notifications to your mobile phone you will require the official [Home Assistant Companion App](https://companion.home-assistant.io/) for iOS or Android installed and configured for your Home Assistant server.

## Automation detail

The single automation I have written can be used for all of the different Discord Webhooks in Vine Helper as it inspects the payload and formats the content accordingly.

## Usage

### Create the Home Assistant Automation

To Create a New Automation with the provided YAML, follow these instructions.

1. Navigate to Automations:

Open your Home Assistant dashboard.

In the left-hand sidebar, click on Settings.

From the Settings menu, select Automations & Scenes.

2. Start a New Automation:

On the Automations screen, click the blue "+ Create automation" button in the bottom-right corner.

A new window will pop up. Select the option "Create new automation" to start with an empty automation.

3. Switch to YAML Mode:

You are now in the automation editor (it defaults to the Visual Editor).

At the very top of the page, click where it says "New automation" click the three-dot menu (⋮) in the top-right corner of the screen.

From the dropdown menu, select Edit in YAML.

Paste Your Code:

The editor will now show a small amount of default YAML (description: "", triggers: [], etc.).

Delete all of the text in this editor box.

Paste the complete YAML code into the now-empty box.

```YAML
# --- Vine Helper Home Assistant Webhook Notifications V1.0 ---
# --- https://github.com/stupot42/VineHelperHAWebhooks ---
# --- Written by Stuart Feltham @stupot42 ---
# --- Last Update: 5/11/2025 ---
# --- Tested with VH Version 3.8.4 ---
#
#
# --- YOU CAN USE THIS ONE WEBHOOK WITH ALL OPTIONS ON THE VH DISCORD TAB WITHOUT CHANGING THE CODE ---
# --- THE EXPECTED JSON PAYLOAD CAN BE FOUND IN THE README ---
# --- IF YOU CHANGE THE PAYLOAD IN VINE HELPER YOU WILL NEED TO UPDATE THE NOTIFICATION ACTION TO TAKE THIS INTO ACCOUNT ---
#
#
alias: "Amazon Vine: Smart Rich notification"
description: "Webhook triggered by VH to notify user of new items"
triggers:
  # --- We need a webhook to trigger our automation from the Vine Helper Settings Discord Tab ---
  - trigger: webhook
    allowed_methods:
      - POST
      - PUT
    local_only: false
    # *** CHANGE THE WEBHOOK ID ***
    # The webhook_id needs to be unique and ideally something that can't be guessed. 
    # It will be accessible to anyone outside your network and should be treated like a password.
    # DO NOT SHARE IT WITH OTHERS
    # Change the contents between the inverted commas.
    # This resource can help generate ids:
    # https://www.guidgenerator.com/online-guid-generator.aspx
    webhook_id: "Abcdefghijklmnopqrstuvwxyz" # <-- Change this!
conditions: []
actions:
  # *** CHANGE YOUR NOTIFY ENTITY ***
  # Change your entity to point to your phone.
  # See guidance 
  - action: notify.mobile_app_iphone # <-- Change this!
    metadata: {}
    data:
      title: "{{ trigger.json.content }}"
      message: "{{ trigger.json.embeds[0].title }}"
      data:
        image: "{{ trigger.json.embeds[0].thumbnail.url }}"
        url: | # <-- Change url to clickAction: if using an Android device
          {% set content = trigger.json.content %}

          {% if content == "New item in RFY!" %}
            https://www.amazon.co.uk/vine/vine-items?queue=potluck

          {% else %}
            {% set title = trigger.json.embeds[0].title %}
            {% set truncated_title = title | truncate(50, killwords=False, end='') %}
            https://www.amazon.co.uk/vine/vine-items?search={{ truncated_title | urlencode }}

          {% endif %}
mode: queued
max: 10
```

4.  Edit the Automation:

Anywhere you see the comment `# <-- Change this!` at the end of a line you should check to see if you need to update the content.

   * Change the Webhook_id
   This is the code that Vine Helper sends to Home Assistant to make it fire the notification.
   I recommend using a GUID generator to create a random string that you can use, eg. [https://www.guidgenerator.com/online-guid-generator.aspx](https://www.guidgenerator.com/online-guid-generator.aspx).

   * Change the device you wish to notify.
   If you aren't sure what your notify service name is, in a new tab open a new Home Assistant dashboard and in the left menu go to __Developer tools__ > __Services__ and type "notify" into the __Service__ drop down box at the top of the page. You will see a list of all the devices you can notify. Make a note of the one you wish to notify. Companion App devices will start with `notify.mobile_app`.
   Update the Automation to notify this device. E.g. `notify.mobile_app_iphone`

   * Check the URL option.
   This code is formatted to work on an iOS device (iPhone or iPad). If you are using an Android device you will need to change `url: |` to be `clickAction: |` in order for the URLs to work. 

5. Note down the webhook URL

Your webhook URL is the address that Vine Helper is going to call when an item is found. The URL is made up of your remote Home Assitant URL, followed by `/api/webhook/` and the webhook_id you chose.

IF your Home Assistant dashboard was normally accessed at:
`https://arandomstring.ui.nabu.casa/lovelace/default_view`
and your webhook_id was: 
`Abcdefghijklmnopqrstuvwxyz`
then your webhook URL would be:
`https://arandomstring.ui.nabu.casa` + `/api/webhook/` + `Abcdefghijklmnopqrstuvwxyz`
or
`https://arandomstring.ui.nabu.casa/api/webhook/Abcdefghijklmnopqrstuvwxyz`

6. Save the Automation:

Once your code is complete, click the Save button (usually in the bottom-right corner).

Your new automation is now created, named "Amazon Vine: Smart Rich notification", and running based on the YAML you provided.

### Update Vine Helper

1. In the Vine Helper settings open the "Discord" tab.

2. Check the boxes for the categories of items you would like to be notified about.

3. Paste the Webhook URL you created above into the "Webhook URL" box for each of your chosen categories. 

### Now Wait

You will now need to wait for a drop to see if your automation works. 

If you're so inclined you can use a programme like Postman to test or `curl` from the command line. 

Alternatively you can add `- PUT` after line 20 of the automation and call the URL from a browser to see if it triggers.


## Important notes

* When you open a notification for the first time, it will open Home Assistant and ask if you want to open the link. If you choose "always open" it will skip straight to the link in future.
* The Webhook has to work remotely. It ***must not be*** local only!
* Do not share your webhook_id or webhook URL with anyone else. Treat it like a password.
* The automation is expecting a JSON payload formatted as follows:
```JSON
{
"content": "New item in RFY!",
"embeds": [{
	"title": "Lucky Rabbit foot",
	"url": "https://www.amazon.co.uk/vine",
	"thumbnail":  {
		"url": "https://m.media-amazon.com/images/I/412LJQa9JrL._SS210_.jpg"
	}
}]
}
```
  If you are a tier 3 Vine Helper supporter and change the JSON you will also need to update the automation accordingly.