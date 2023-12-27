# 360 Robot Vaccum Reverse Engineering

ToC:
- [Authentication](#authentication)
- [Sending Commands](#sending-commands)
- [Home Assistant](#home-assistant)


# Authentication

A QID and SID, and vacuum SN are required. (User and session IDs)

## Obtaining the QID:
This is your user account number:

1. Sign into the 360 website using this link: [https://i.360.cn/reg/?src=pcw_open_app&destUrl=http%3A%2F%2Fdev.app.360.cn%2Fapp%2Flist](https://i.360.cn/login/?src=pcw_home&destUrl=https://www.360.cn/)
2. Visit [http://open.app.360.cn/developer/](http://open.app.360.cn/bbs/index) and copy the ten digit number beginning in `360Uxxxxxxxx` at the top right. This is your QID. It should look like this: `3123795435`

## Obtaining the SID:

~~The SID is a 32-character long session ID used to authenticate HTTP requests to the API. Obtaining this key is currently non-trivial and will be documented after a better method is found for extracting it.~~

If you have an Android phone or emulator with ADB setup, you can run the following command on your PC to dump the keys from the app's logs. (facepalm!) It may take a moment, to speed it along you can interact with the 360 app. 

```bash
adb logcat | grep -m 1 MyPushMessageListener.java | tail -c 93
```

It will output the QID, SID, and the Push Key. Save the push key, as this may be useful in the future. It is required for some data like battery status that uses the push server.


## Generating a cookie

Once you have your SID and UID, you can use the following cookie to authenticate:

```text
q=u=&t=1;t=&v=2.0&a=1; qid=1234123412; sid=4a20145c62428d31b52b53c9ccbfcee4
```
## Obtaining the SN:

The SN can be viewed in Toolbox->Settings->Device Info on the app. You can also use the API to get a list of vaccums assosciated with your account:

```bash
curl -X POST \
  https://q.smart.360.cn/common/dev/GetList \
  -H 'Cookie: q=u=&t=1;t=&v=2.0&a=1; qid=1234123412; sid=4a20145c62428d31b52b53c9ccbfcee4' \
```


# Sending commands

With the above info you can now send arbitrary commands to the vacuum. Most commands are send to the `clean/cmd/send` endpoint. Here is an example of the "Find Robot" command, which makes it speak. 

```bash
curl -X POST \
  https://q.smart.360.cn/clean/cmd/send \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'Host: q.smart.360.cn' \
  -H 'Connection: Keep-Alive' \
  -H 'Accept-Encoding: gzip' \
  -H 'Cookie: q=u=&t=1;t=&v=2.0&a=1; qid=1234123412; sid=4a20145c62428d31b52b53c9ccbfcee4' \
  -d 'sn=361TY*******542&infoType=21020&data=%7B%22ctrlCode%22%3A3010%7D&devType=3'
```
```text
POST /clean/cmd/send HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 234
Host: q.smart.360.cn
Connection: Keep-Alive
Accept-Encoding: gzip
Cookie: q=u=&t=1;t=&v=2.0&a=1; qid=1234123412; sid=4a20145c62428d31b52b53c9ccbfcee4

sn=361TY*******542&infoType=21020&data=%7B%22ctrlCode%22%3A3010%7D&devType=3
```

# Main API Command List

## clean/cmd/send
The following `infoType` values have been tested:

Name | infoType | Request Data | Recieve Data | Notes
--- | --- | --- | --- | ---
Clean | 21005 | {"mode":"smartClean","globalCleanTimes":1} | None |
Recharge | 21012 | {"cmd":"start"} | None | 
Pause | 21017 |  {"cmd":"pause"} | None |
Heartbeat | 21006 | {"heartbeatSec":60} | None | Unsure what exactly it's used for. Doesn't appear to cause any issues if it isn't sent.
Set Clean Mode | 21022 | {"cmd":"quiet", "cleanType":"total"} | None | cmd one of ["quiet","auto","strong","max"]
Set LED mode | 21024 | {"cmd":"setledswitch","value":0} | None | 
Set Avoid Walls | 21024 | {"cmd":"setSoftAlongWall", "value":1} | None |
Find Robot | 21020 | {"ctrlCode":3010} | None |
Unknown 1 | 21011 | {"startPos":0,"userId":"0","mask":0} | None | Unknown. Sent regularly during cleaning.
Unknown 2 | 21015 | None | None | Sent once on boot. Unknown. Doesn't appear to cause any issues if it isn't sent.

The following have not yet been tested, use at your own risk:

Name | infoType | Request Data | Recieve Data | Notes
--- | --- | --- | --- | ---
| setRemoteControlNet | 21037 | [str] | None | Possibly related to enabling the UDP interface |
| reboot | 21024 | {"cmd":"reboot","value": [i2]} | None | Logs reboot task with specified delay in milliseconds |
| getWifiInfo | 21019 | None | Unknown |

A full list of all known infoTypes is available in the infoType_fields.md file.

Command `30000` is unknown, may be related to "composite cmds" 


Where "None" is specified as a data type, the following is expected: 

```json
{
  "errno": 0,
  "errmsg": "Succeeded",
  "data": {}
}
```

A full list of error codes is available here: https://smart.360.cn/clean/errorInfo_us.json


# UDP API

There is a UDP RC API on port 8790. It is normally closed, and opened when RC mode is enabled. It operates similarly to the Main API. It is not yet known if normal infoType commands can be sent here; this could allow local control. All known commands use infoType 20120

Sending movement commands: `{"infoType":21020,"data":{"ctrlCode":3013,"ctrlParams":{"speedV":0.000000,"speedW":0.000000}},"packId":27}`

Reporting current position: `{"message":"OK","infoType":21020,"x":-261,"y":-392,"angle":1410,"packId":27}`

Unknown, possibly to close connection: `{"infoType":21020,"data":{"ctrlCode":4000},"packId":194}`

Reversing the android app indicates a generic UDP API that appears to be able to recieve other commands, but this isn't supported on my vacuum. 

# Push API

There is another API at  that appears to be used by the app for for higher-traffic uses including battery status and mapping data. As sending commmands was my primary goal this has not been explored. On my device it shows traffic to `101.198.193.215`, which does not reverse to a DNS name. 

The data on this API is secured by a symmetric AES-128 key. 

Might just be a proxy for data directly from the vac, stuff that the main API doesn't care about. I would like to get status info to integrate with Home Assistant, so this API is probably next.

## Decrypting Push API packets

The Push API uses a wonky AES implementation; you need to convert the key to ASCII hex, read it as UTF-8, and use the first 16 bytes as both the key and IV. Here is an example of a decrypted packet:

```text
{"createTime":"1703612364","data":"{\"message\":\"ok\",\"infoType\":20001,\"data\":{\"allArea\":5943,\"allTime\":445962,\"autoBoost\":0,\"cleanArea\":3,\"cleanId\":\"361TY000003000000-1703612364\",\"cleanTime\":50,\"elec\":89,\"elecReal\":89,\"errorState\":[0],\"errorTime\":0,\"lastSubMode\":\"total\",\"led\":0,\"mode\":\"charge\",\"mopStatus\":0,\"phi\":-1953,\"pos\":[540,-483],\"reliable\":1,\"showSmartArea\":0,\"showSweepArea\":0,\"soft\":1,\"subMode\":\"null\",\"timeStamp\":1703612364,\"timerStatus\":10,\"vol\":9,\"windPower\":0,\"workNoisy\":\"quiet\"}}","event":4,"sn":"361TY000003000000","taskid":"1703612364234672"}
```


I don't yet know how to initiate a connection to the push API server. 

# Main API endpoints:

This is a list of endpoints which is used by the 360 mobile app to control 360 smart ai devices like 360 S5 / S6 / S7 vacuum cleaner.


### clean
https://q.smart.360.cn/clean/cmd/send

Most of the commands are sent here. Includes heartbeat, sweep, voice, notify, etc.

https://q.smart.360.cn/clean/devuser/updateInfo

https://q.smart.360.cn/clean/record/statis

Returns total lifetime cleaning statistics for a given serial number, and the human readable string (data.clues) used on the Cleaning Report page of the app. 

```json
{
  "errno": 0,
  "errmsg": "Succeeded",
  "data": {
    "cleanArea": 594,
    "cleanCount": 22,
    "cleanTime": 44175,
    "clues": "The accumulated cleaning area is equivalent to ((4.1)) basketball courts, saving you ((12.3)) hours",
    "mopArea": 0
  }
}
```


https://q.smart.360.cn/clean/record/allStatis

Similar to `clean/record/statis`, just returns seconds instead of hours. 

```json
{
  "errno": 0,
  "errmsg": "Succeeded",
  "data": {
    "allCleanArea": 5183,
    "allCleanCount": 662,
    "allCleanTime": 833558,
    "allMopArea": 0,
    "bindNum": 1
  }
}
```



https://q.smart.360.cn/clean/record/getList

Returns an array of recent cleaning sessions, including time, modes, statistiscs and success. Indexed by `data.list[].cleanId`, which is a string in the format `${Serial}-${Timestamp}`, (which appears to match `data.list[].startTime`)

https://q.smart.360.cn/clean/record/recentlycleanlist

Returns monthly and weekly history of square meters cleaned. Used on the Cleaning Report page of the app. 

https://q.smart.360.cn/clean/record/getOne
https://q.smart.360.cn/clean/ad/applist

https://q.smart.360.cn/clean/dev/getMaterialStatus

Unknown. No data is sent or recieved, just "Succeeded". Appears to be accessed once on first launch. 


https://smart.360.cn/clean/modelalias.json

Returns a list of all the vaccum models with some OTA and naming data. Has flags (ovLogReport and cnLogReport) that appear to be whether log reporting is enabled on overseas servers and domestic servers

https://smart.360.cn/clean/errorInfo_us.json

No authentication required. Returns a list of error codes and localized (en_us) strings. 





### map
https://q.smart.360.cn/clean/record/getBackupMapList
https://q.smart.360.cn/s3_file_new/iot-master-clean-online-pub1-bjyt

### json
https://smart.360.cn/clean/errorInfo_us.json
https://smart.360.cn/clean/share_act.json
https://smart.360.cn/clean_dev/list_en_us.json

### sdk
https://sdk.s.360.cn/ak/e995f98d56967d946471af29d7bf99f1.html?m2=8D7ACEB0-8EA1-4AD5-9339-34D01918FC4A

### update
https://p.s.360.cn/update/update.php

### upgrade
https://ota3.jia.360.cn/upgrade/getNewVersion

### statistics
https://p.s.360.cn/pstat/plog.php

### common
https://q.smart.360.cn/common/dev/GetList

Returns a list of vaccums assosciated with an account. Authentication required.

```json
{
  "errno": 0,
  "errmsg": "Succeeded",
  "data": {
    "list": [
      {
        "sn": "[redacted]",
        "devType": 3,
        "icon": "https://p.ssl.qhimg.com/t01047fcf6ee991aeb9.png",
        "title": "Example Vacuum",
        "hardware": "S7",
        "pkcode": "",
        "hwSn": "",
        "version": "1.8.9",
        "versionCode": 2976,
        "role": 2,
        "online": 1,
        "support": "{\"carpetMode\":1,\"cleanTimesInTimer\":1,\"deleteMap\":1,\"led\":1,\"logReport\":1,\"maxBanArea30\":1,\"maxMode\":1,\"mop\":1,\"mopBanArea\":1,\"multiQuietHours\":1,\"reboot\":1,\"remoteControl\":1,\"remoteControlWithMap\":1,\"restoreMap\":1,\"roomSweep\":1,\"rotateMap\":1,\"rssi\":1,\"saveMap\":1,\"setRoomAttrib\":1,\"smartArea\":1,\"softAlongWall\":1,\"sweepAreaInTimer\":1,\"timedSweepMode\":1,\"virtualWall\":1,\"voicePacket\":1,\"volume\":1}",
        "alexaDefault": 0,
        "ownerQid": 0,
        "ownerImage": "",
        "ownerNickName": "[redacted]",
        "bindTime": 0
      }
    ]
  }
}
```



https://q.smart.360.cn/common/share/getInviteList
https://q.smart.360.cn/common/share/inviteByCode
https://q.smart.360.cn/common/share/getInviteList

### product images
http://p0.qhimg.com/t0123a732c50f157afd.png


# Home Assistant

A minimal working home assistant config. Place the cookie contents in a variable called `360_token` in `./secret.yaml`

```yaml
rest_command:
  vacuum_pause_button:
    url: "https://q.smart.360.cn/clean/cmd/send"
    method: "post"
    headers:
      Content-Type: "application/x-www-form-urlencoded"
      Host: q.smart.360.cn
      Accept-Encoding: gzip
      Cookie: !secret 360_token
    payload: "sn=361TY000000000000&infoType=21017&data=%7B%22cmd%22%3A%22pause%22%7D&devType=3"
  vacuum_charge_button:
    url: "https://q.smart.360.cn/clean/cmd/send"
    method: "post"
    headers:
      Content-Type: "application/x-www-form-urlencoded"
      Host: q.smart.360.cn
      Accept-Encoding: gzip
      Cookie: !secret 360_token
    payload: "sn=361TY000000000000&infoType=21012&data=%7B%22cmd%22%3A%22start%22%7D&devType=3"
  vacuum_clean_button:
    url: "https://q.smart.360.cn/clean/cmd/send"
    method: "post"
    headers:
      Content-Type: "application/x-www-form-urlencoded"
      Host: q.smart.360.cn
      Accept-Encoding: gzip
      Cookie: !secret 360_token
    payload: "sn=361TY000000000000&infoType=21005&data=%7B%22mode%22%3A%22smartClean%22%2C%22globalCleanTimes%22%3A1%7D&devType=3"
  vacuum_find_button:
    url: "https://q.smart.360.cn/clean/cmd/send"
    method: "post"
    headers:
      Content-Type: "application/x-www-form-urlencoded"
      Host: q.smart.360.cn
      Accept-Encoding: gzip
      Cookie: !secret 360_token
    payload: "sn=361TY000000000000&infoType=21020&data=%7B%22ctrlCode%22%3A3010%7D&devType=3"
```

# Notes

There's a few abbreviations used in their logging: 

"jioe" is JSONIOException, "ste" is SocketTimeoutException, and 


# Android App Packages:

`com.qihoo.smarthome.sweeper.ui.o1`: RC mode, UDP reciever

