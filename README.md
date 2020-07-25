## Servicenow Script Editor Engine (sn-edit.com)

## Maintainers note
This repository is now archived, please see [sn-edit](https://github.com/sn-edit/sn-edit) for the new version! 
This repository will not be maintained anymore!


Did you ever wanted to edit the scripts, ui pages, html etc. locally in your favorite IDE, like Webstorm, Visual Studio Code or others? 

Well, this is your chance to try. This app will allow you to do just that. With minimal footprint, quick reaction time to changes, you will be able to edit everything locally and sync back to the instance right after saving the file.

The app works without you being interacting with it. You need to run it, edit your scripts and after save the app will automatically detect this through file system changes and upload the script to the instance right away.

The code is for now not Open Source but this might change in the future. The app runs with zero dependency. All you need to do is download the latest release, setup the config file according to your custom instance and run the app.

# How does this work anyway?

The flow is right now very simplistic. The app syncs the business rules and script includes froom the instance to two folders. All of the files are named based on this pattern `SysName_SysId.js`. It is important to keep the original name. After downloading all the scripts, we set a watcher on these two folders to listen for fs (filesystem) events and if something is saved or written to, then we upload these to the instance. Right now the app does not chevck for conflicts or newer versions on the instance, it just simply uploads the changes regardless of the newer versions, but here is where I need yuo, the potential user, so we can think of a simple and straightforward way on how to do these collision checks and improve the experience. If you delete on or more scripts from your disk, those will be reacquired on the next run.

# Downloads
Here you will always find a link to the latest version of the app which you can download and run. All previous releases can be found on the [releases](https://github.com/0x111/sn-edit.com/releases) page.

Every new release will be published with a precompiled binary for Linux, Mac OS (darwin) and Windows platforms with a 64 bit binary. No support for 32 bit versions right now. If there will be a broad requirement, there can be other platforms included, like for example ARM for Raspberry PI or Android maybe.

To check the downloaded binary for validity, you can use the checksums.txt for compare. We do not share these elsewhere except of github and this repository. Due to this, to be sure you don't download something harmful, always use this source.

## Download the latest version below
[sn-edit v0.1.2](https://github.com/0x111/sn-edit.com/releases/latest)

# Getting Started
After you download the executable file, you can feel free to put it anywhere you would like to. The basic requirement is, that your user or whatever user you will run this with has access to the filesystem and the correct permissions to the config file you will create in the next steps and of course to the locations where the scripts will be downloaded.

## Config file
The config file is quite extensive, very broad and at first it may sound complicated, but I assure you that it can be easily setup. I will provide a short example and we go through it thoroughly to have a basic feel of the options and what they mean exactly.

```json
{
  "db_path": "/Users/username/sn-edit/scripts.db",
  "log_level": "info",
  "rate_limiting_clearing": 2,
  "rest_credentials": {
    "password": "password",
    "user": "user",
    "xor_key": "randomxorkey",
    "masked": false
  },
  "script_path": "/Users/username/path/to/scripts/folder/",
  "servicenow_instance_url": "https://devxxxx.service-now.com/",
  "tables": [
    {
      "fields": [
        {
          "field": "sys_id"
        },
        {
          "field": "script"
        },
        {
          "field": "sys_name"
        }
      ],
      "table": "sys_script_include"
    },
    {
      "fields": [
        {
          "field": "sys_id"
        },
        {
          "field": "script"
        },
        {
          "field": "sys_name"
        }
      ],
      "table": "sys_script"
    }
  ]
}
```

#### `db_path`

This specifies the db path, it can be a relative path, or a full path, but I would recommend to use a full path. In this database we will store some information about the scripts synced with this app, like sys_id, sys_name and the file checksum. Basically what we use from this is actually minimal at the moment. Right now it has an informational character, but later on could be used for various purposes.

#### `log_level`

If you will be just using the app, you will mostly need the info level, if you encounter a problem and want to open an issue, you will need to set this to debug and see the output of the various events/processes so we can help you solve your problem. Attention! The `debug` level will produce a lot of output, so use it wisely, for informational and basic usage, please just use the `info` level.

#### `rate_limiting_clearing`

We needed an option for this too, like already described, the app listens on fs events and uploads the changes to the instance. Some editors like VSCode for example create 2x write events after saving a file, this was causing double uploads to the instance. So the rate limiting is here to put a brake on it. The default recommended value is 2 seconds. This means that in one specific moment, we will only upload the changes once during a two seconds time period. After two seconds the rate limiting get's reset and you can work further.

#### `rest_credentials`

These credentials are the ones used to download and/or upload the scripts to/from the instance. This is basically a user, which has the rights to use the Web Services on the Instance and has the correct access rights to the tables. While developing the app, for ease of access, we used a test user with admin rights on the instance, but this is managed out of the scope of the app.

Since there were requests at my previous work for the Jetbrains IDE, to mask the password in some way so it is not just sitting there in a plaintext, then there is an option for this here already.
```json
"rest_credentials": {
    "password": "password",
    "user": "user",
    "xor_key": "randomxorkey",
    "masked": false
  }
```
  At the first run, fill out your username and password as usual, set the `masked` value to false, set a random `xor_key` per your liking, this will be only stored in this config file and nowhere else. After the first run, the app will mask your password and will update the value in the config file. So after you run the app and then look at your config file you will see some kind of gibberish text instead of your password. This way we will not store the plaintext value longer than the first run. It is important to not change the xor_key afterwards, because then the app won't be able to decrypt it and the rest calls will fail. If you would like to update the password or change the key, you can do that by changing the password value to your new password, change the `xor_key` if needed and then set the `masked` value to false. This way the app will know to mask the password again after the next run.
  
#### `script_path`
You need to set a full path here to the directory, where the scripts should be stored on your disk. On a Mac it would look like in the sample config file, but you can set it similarily on every platform. It is important that it is the full route to your scripts folder. This folder needs to be writable by the user running the app. If there are permission problems, please use the `debug` log level to diagnose. *Attention: The script path needs to end with a trailing slash `/`. This is required by design.*

#### `servicenow_instance_url`
This is a full url to your servicenow instance. It can be a dev instance from servicenow directly or some customer instance.
*Attention: The servicenow instance url needs to end with a trailing slash `/`. This is required by design.*

#### `tables`
This block describes the tables which we would like to sync from the API. Right now, the app works with two tables, you can define more, but there can be unforseen side effects. We would like to go with the two mostly important tables for scripting here and we will see from there.

Example:
```json
"tables": [
    {
      "fields": [
        {
          "field": "sys_id"
        },
        {
          "field": "script"
        },
        {
          "field": "sys_name"
        }
      ],
      "table": "sys_script_include"
    },
    {
      "fields": [
        {
          "field": "sys_id"
        },
        {
          "field": "script"
        },
        {
          "field": "sys_name"
        }
      ],
      "table": "sys_script"
    }
  ]
```
The `tables` property is an array of objects here. Every object, contains two properties:
* fields - this is an array of objects, denouncing the fields which we need to acquire from the API. Please define all the mentioned fields, all three of them are required for this app to work correctly. So we need sys_id, script and sys_name. 
* table - this is the table name which we want to sync with the app

# Running the app
If you filled out the config file as described above, you are ready to launch the app. For the following examples, we will assume that the binary is called `sn-edit`:

- Run help

To output the help text for the program, run the --help command.
```bash
sn-edit --help
```
Also for the outputted commands like version or sync, you can run a --help too. So if you would like to know what parameters the `sync` command accepts, you can run this command:
```bash
sn-edit sync --help
```
This will output additional flags for the program which can modify it's behavior.

If you do not wan't to do anything fancy, you can simply run the following command to start the app:
```bash
sn-edit sync
```

The app will search for a config file with the name `servicenow-script-editor.json`. It will do so either in the current directory or in the current users home directory.

If for some reason you do not like the name, you can override the config file's name with the following command. Let's say that we wan't to name our config file `custom_config.json`.
```bash
sn-edit sync --config custom_config
```
Right now, the app would search for a file named `custom_config` in the aforementioned directories to load the config from. You do not need to specify an extension here, it does automatically search for the file.

#### Issues
For issues please use the issue tracker on github (Follow this link: [Issues](https://github.com/0x111/sn-edit.com/issues)). While describing your issue, please be as precise as you can be on what you did or did not do to encounter the issue. Provide all steps which are needed to reproduce the issue so we can help you in your case. 

#### Disclaimer
The app does not collect any data, username, password or any other sensitive information knowingly. This app does not need any admin permission or any installation, so all you need to run it is to download it, set it up and then use it while developing new functionality for servicenow. This software is provided 'AS IS', we are not responsible for any damages caused by using this app while you work on your project or instance. Use this with caution and at your best judgement.
