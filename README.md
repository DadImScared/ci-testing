
# Configuring the Firebase Cloud Functions environment!
The main credentials, **databaseURL** & the **storageBucket** are automatically set up for you, however, other variables have to be set manually before deploying the functions.
  
```sh
# each arg must have at least 2-part key (e.g foo.bar)
$ firebase functions:config:set website.base_url="Base url of the website"
$ firebase functions:config:set send_in_blue.key="sendInBlue secret Key"
# Coordinator details
$ firebase functions:config:set sqr.allotment.templateid=NUMBER
$ firebase functions:config:set coordinator.email_address='EMAIL'
$ firebase functions:config:set coordinator.timeZoneOffset=NUMBER of HOURs
#Importing a spreadsheet to the database variables
$ firebase functions:config:set sqr.spreadsheetId='Google Spreadsheet ID'
```

Firebase Cloud Functions is written in **TypeScript**, if you are uploading the functions for the first time make sure you're selecting the language used in the project as **TypeScript** instead of the default **JavaScript**.

# Deploying Firebase Cloud Functions!
Don't upload the functions manually as it needs first to be converted from **TypeScript** into **JavaScript**.


The following command will run a **predeploy** script to make the conversion and then upload the functions.


```sh
$ npm run deploy
```

# DB Rules!
In order for the sound editing function to run properly, you have to add the following under the main `rules` object in the **DB rules** page.

```
"sound-editing": {
    "restoration": {
        "allotments": {
            ".indexOn": ["assignee/emailAddress"]
        },
    }
}
```
