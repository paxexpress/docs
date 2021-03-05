# Using of pax.express

The PAX.express service is used for the delivery of firmware to esp32 paxcounter devices. This is done by using the OTA feature of 
the paxcounter project. Our fork of the repo can be found [here](https://github.com/paxexpress/ESP32-Paxcounter). The PAX.express cli can be found [here](https://github.com/paxexpress/paxexpress-cli). Both repositories are used in the discussed setup guide later one. Feel free to clone them for use.

## Setup guide
### Tooling 

Please clone the [paxexpress-cli](https://github.com/paxexpress/paxexpress-cli) to your filesystem. We use poetry for python package management so please also intall [poetry](https://python-poetry.org/docs/#installation).

Using poetry change dirctory to the repo's root directory and call:
```
poetry install
```

after this you should get the cli's command options by using

```
peotry run paxexpress --help
```
### Registration and login
It's a good idea to switch to the poetry shell now:

```
peotry shell
```

Using `auth` module you can use the register command to register to the PAX.express service.

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
PAX.express uses the structure of repositories which contain packages
which again have different versions.

At first you should create a repository to store your packages.

``` 
paxexpress repository create
```

You will be asked for a repository and some description.

After setting the name you can use this name to reference the repository in further commands.

``` 
paxexpress package create -r YourChoosenRepoName
```

e.g. This command creates a package relative to your repository.
Also here your asked for a name and description.

After this steps your read to upload your first firmware.

## Flashing and OTA
You will need the platformio tools to proced with this guide. You can get those e.g. by installing them with your package manager of choise. How to work with platformio is out of scope of this guide.

Lets shift our focus to the firmware repo [here](https://github.com/paxexpress/ESP32-Paxcounter). First you need to make copies of `platformio_orig.ini` naming it `platformio.ini`. In this file you can adjust the Board section and comment in your used `board`.

You also have to create a package on paxexpress by using the cli for the specific boardname. E.g commenting in `olimexpoeiso.h` you would have to have the package `olimexpoeiso` on your repository in paxexpress to store the versions for this specific board class.

Also have a look at `platformio` section. Here you can choose the `default_envs` to be `ota` for uploading to PAX.express or `usb` for flashing your device localy.

In the first step we us local flashing. Now please make a copy of `src/paxcounter_orig.conf` and rename it to `src/paxcounter.conf`.

In this file you configure your paxcounter. Please adjust the values accordingly to your wished configuration. Be aware that you will need to receive rcommands at the later part of this guide. E.g. this can be done by using mqtt and a ethernet port.

Now please make a copy of `src/loraconf_sample.h` and rename it to `src/loraconf.h`. Also  make a copy of `src/ota_sample.conf` and rename it to ``src/ota.conf`. In the later file change `BINTRAY_USER` to your user (not your email you using for login!) on PAX.express, `BINTRAY_REPO` to your created Repositorys' name and `BINTRAY_API_TOKEN` to your password.

Also `OTA_WIFI_SSID` and `OTA_WIFI_PASS` needs to be set to a network the OTA will downloaded over. BE AWARE THIS WIFI CREDENTAILS WILL BE UPLOADED TO PAXEXPRESS INSIDE THE FIRMWARE.

Now you are ready to flash the firmware on your board by using platformio tools.

```
pio run -t upload 
```

When you want to upload a version of the firmware for OTA set the
`default_envs` to be `ota` in `platformio.ini`. Now you can use the same commmand to upload it to PAX.express, but you should increment the `release_version` in `platformio.ini` under `common`.

```
pio run -t upload 
```

Now when sending a rcommand of `[0x9, 0x9]` (or `CQk=` on a mqtt publish) over your rcommand channel of choice a OTA should be triggered on the device.

When downgrading versions be aware to earse the flash beforehand
while being on the env `usb` with

```
pio run -t erase 
```
