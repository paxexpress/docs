# Using of PAX.express

The PAX.express service is used for the delivery of firmware to esp32 paxcounter devices. This is done by using the OTA feature of the [ESP32-paxcounter project](https://github.com/cyberman54/ESP32-Paxcounter). Our fork can be found [here](https://github.com/paxexpress/ESP32-Paxcounter), which is already setup to use PAX.express to deliver firmware updates. The PAX.express cli can be found [here](https://github.com/paxexpress/paxexpress-cli).

## Setup guide
### Tooling 

Please clone the [paxexpress-cli](https://github.com/paxexpress/paxexpress-cli) to your filesystem. 
For installation informations read the [README.md](https://github.com/paxexpress/paxexpress-cli/blob/master/README.md).

### Registration and login
After installation you can use either `./paxexpress` or `paxexpress.exe` depending of your platform. Substitute the command in the following guide accordingly to your enviroment.

Using `auth` module of the paxexpress-cli you can use the register command to register to the PAX.express service.

``` 
paxexpress auth register
``` 

You will need the provided beta key at this step.

After registration you can use the login command to get a 30min token for auth against your account which is stored in your operating system's password store. 
``` 
paxexpress auth login
```

When you want to remove your auth informations use. 

``` 
paxexpress auth logout
```

### Repositories and packages
PAX.express uses the structure of repositories which contain packages which again have different versions.

At first you should create a repository to store your packages.

``` 
paxexpress repository create
```

You will be asked for a repository and some description.

After setting the name you can use this name to reference the repository in further commands e.g.

``` 
paxexpress package create -r YourChoosenRepoName
```

This command creates a package relative to your repository.


After this steps your read to prepare the upload of your first firmware.

## Flashing and OTA
You will need the platformio tools to proced with this guide. You can get those e.g. by installing them with your package manager of choise. How to work with platformio is out of scope of this guide.

Lets shift our focus to the firmware repo [here](https://github.com/paxexpress/ESP32-Paxcounter). First you need to make copies of `platformio_orig.ini` naming it `platformio.ini`. In this file you can adjust the Board section and comment in your used `board`.

You also have to create a package on paxexpress by using the cli for the specific boardname. E.g commenting in `olimexpoeiso.h` you would have to have the package `olimexpoeiso` on your repository in paxexpress to store the versions for this specific board class.

Use this cli command and set the name of the package to the boardname you choose:
``` 
paxexpress package create -r YourChoosenRepoName
```

Also have a look at `platformio` section. Here you can choose the `default_envs` to be `ota` for uploading to PAX.express or `usb` for flashing your device localy. In the first step we us local flashing, there for we use the `usb` `default_envs`.

Now please make a copy of `src/paxcounter_orig.conf` and rename it to `src/paxcounter.conf`.

In this file you configure your paxcounter. Please adjust the values accordingly to your wished configuration. Be aware that you will need to receive rcommands at the later part of this guide. E.g. this can be done by using mqtt and a ethernet port.

Now please make a copy of `src/loraconf_sample.h` and rename it to `src/loraconf.h`. Also  make a copy of `src/ota_sample.conf` and rename it to `src/ota.conf`. In the later file change `BINTRAY_USER` to your user (not your email you using for login!) on PAX.express, `BINTRAY_REPO` to your created Repositorys' name and `BINTRAY_API_TOKEN` to your password.

Also `OTA_WIFI_SSID` and `OTA_WIFI_PASS` needs to be set to a network the OTA will downloaded over. BE AWARE THIS WIFI CREDENTAILS WILL BE UPLOADED TO PAXEXPRESS INSIDE THE FIRMWARE AND ARE AVALIBLE PUBLICLY ON THE SERVICE AT THIS POINT.

Now you are ready to flash the firmware on your board by using platformio tools.

```
pio run -t upload 
```

When you want to upload a version of the firmware for OTA set the
`default_envs` to be `ota` in `platformio.ini`. Now you can use the same commmand to upload it to PAX.express, but you should increment the `release_version` in `platformio.ini` under `common`.

```
pio run -t upload 
```

Now the rcommand to trigger OTA can be used: `[0x09 0x09]` via LoRa or `CQk=` via mqtt

When downgrading versions be aware to earse the flash beforehand
while being on the env `usb` with

```
pio run -t erase 
```
