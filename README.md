# 360 Robot Vaccum Reverse Engineering

ToC:
1. [Authentication](#authentication)
1. [Sending Commands](#sending-commands)

2. 
# Authentication

A QID and SID, and vacuum SN are required. (User and session IDs)

## Obtaining the QID:
This is your user account number:

1. Sign into the 360 website using this link: [https://i.360.cn/reg/?src=pcw_open_app&destUrl=http%3A%2F%2Fdev.app.360.cn%2Fapp%2Flist](https://i.360.cn/login/?src=pcw_home&destUrl=https://www.360.cn/)
2. Visit [http://open.app.360.cn/developer/](http://open.app.360.cn/bbs/index) and copy the ten digit number beginning in `360Uxxxxxxxx` at the top right. This is your QID. It should look like this: `3123795435`

## Obtaining the SID:

The SID is a 32-character long session ID used to authenticate HTTP requests to the API. Obtaining this key is currently non-trivial and will be documented after a better method is found for extracting it. 

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
  http://q.smart.360.cn/clean/cmd/send \
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
| addNewWifi | 21028 | {"cmd":"addWifi","wifiName":"[str]","wifiPwd":"[str2]"} | None | |
| getWifiInfo | 21019 | None | None |
| changeMap | 21025 | {"cmd":"changeMap","cleanId":"[str]","downUrl":"[str2]","md5":"[str3]"} | None | |
| continueSweep | N/A | UUID.randomUUID().toString() | None | |
| deleteMap | 21024 | {"cmd":"cancleMap","value":0} | None | Logs deletion task with a UUID |
| deleteWifi | 21028 | {"cmd":"deleteWifi","wifiName":"[str]"} | None | |
| getAutoUpdateData | 21034 | "" | None | |
| reboot | 21024 | {"cmd":"reboot","value": [i2]} | None | Logs reboot task with specified delay in milliseconds |
| setAreaSplitFinish | 21038 | {"cmd":"confirm","type":"[str]"} | None | Logs area split confirmation task with a UUID |
| setAutoForbidArea | 26000 | {"cmd":"[z ? "set" : "cancel"]","mapId":[j],"smartForbidAreaId":[i2]} | None | |
| setAutoUpdateFirmware | 21024 | {"cmd":"autoUpdate", "value":[i2],"timeZone":[TimeZone.getDefault().getOffset(System.currentTimeMillis()) / 3600000.0f]} | None | Logs firmware auto-update task with specified value |
| setAutoWater | 21024 | {"cmd":"setAutoWater", "value":[i2], "cleanType":"[str]"} | None | Logs auto water task with specified value and clean type |
| setAvoidFallingDown | 21024 | {"cmd":"setAvoidFallingDown", "value":[i2]} | None | Logs avoid falling down task with specified value |
| setBatteryProtection | 21024 | {"cmd":"setBatteryProtection","value":[z ? 1 : 0]} | None | Logs battery protection task with a UUID |
| setBlockSweep | 21024 | {"cmd":"setBlockSweep","leftValue":"[str]","rightValue":"[str2]"} | None | Logs block sweep task with specified left and right values |
| setCarpetAuto | 21024 | {"cmd":"carpetAutoRecognize","value":[z ? 1 : 0]} | None | Logs carpet auto recognition task with a UUID |
| setCarpetDepthOn | 21024 | {"cmd":"setCarpetDepthClean","value":[z ? 1 : 0]} | None | Logs carpet depth clean task with a UUID |
| setCarpetMopOn | 21024 | {"cmd":"setCarpetMop","value":[z ? 1 : 0]} | None | Logs carpet mop task with a UUID |
| setCarpetOn | 21024 | {"cmd":"setAutoBoost","value":[z ? 1 : 0]} | None | Logs carpet on task with a UUID |
| setCarpetOrCliffOperate | 21031 | {"cmd":"[str]","type":"[str2]","mapId":[j]} | None | |
| setCurrentWifi | 21028 | {"cmd":"useWifi","wifiName":"[str]"} | None | |
| setHallwaySweepTwoCountOn | 21024 | {"cmd":"hallwaySweepTwoCount","value":[z ? 1 : 0]} | None | Logs hallway sweep two count task with a UUID |
| setKitchenToiletLastSweepOn | 21024 | {"cmd":"kitchenToiletLastSweep","value":[z ? 1 : 0]} | None | Logs kitchen toilet last sweep task with a UUID |
| setLedOn | 21024 | {"cmd":"setledswitch","value":[z ? 1 : 0]} | None | Logs LED on task with a UUID |
| setMaterial | 21024 | {"cmd":"roomMaterial","value":[z ? 1 : 0]} | None | Logs material task with a UUID |
| setMopOnly | 21024 | {"cmd":"setMopSwitch","value":[z ? 2 : 1]} | None | Logs mop mode task with a UUID |
| setQuicklyMap | 21036 | {"mode":"quicklyMap"} | None | |
| setRemoteControlNet | 21037 | [str] | None | |
| setSoftAlongWall | 21024 | {"cmd":"setSoftAlongWall", "value":[i2]} | None | Logs soft along wall task with specified value |
| setSweepAreas | N/A | "21023" and "21005" commands with sweep area enable and sweep command | None | Logs setting sweep areas task with specified areas and temp areas |
| setSweepMode | 21022 | {"cmd":[SweepMode.getMode(i2)]} | None | Logs setting sweep mode task with specified mode |
| setVolume | 21024 | {"cmd":"setVolume","value":[i2]} | None | Logs setting volume task with specified value |
| setWaterPump | 21024 | {"cmd":"setWaterPump", "value":[i2]} | None | Logs setting water pump task with specified value |
| startAreaSweep | N/A | Various commands based on conditions | None | Logs starting area sweep task with specified areas, continue flag, task ID, submode, clean times, and active ID |
| startBlockAreaSweep | 21032 | {"cmd":[str],"type":[str2],"mapId":[getMapId()],"areas":[list2],"taskid":[str3]} | None | Logs starting block area sweep task with specified mode, areas, type, task ID, and active ID |
| startCharging | 21012 | {"cmd":"start"} | None | Logs starting charging task with a UUID |
| startEdgeSweep | 21005 | {"mode":"edgeClean"} | None | |
| startPointSweep | 21005 | {"mode":"pointClean", "count":[i2], "style":[i3]} | None | |
| startRemoteControl | 21026 | "" | None | |
| startRemoteControlNet | 21026 | {"type":"tcp"} | None | |
| startSmartAreaSweep | 21005 | {"mode":"smartArea"} | None | Logs starting smart area sweep task with a UUID |
| startSmartSweep | 21005 | {"mode":"smartClean", "globalCleanTimes":[i2]} | None | Logs starting smart sweep task with a UUID and global clean times |
| startSweep | N/A | "21023" and "21005" commands with sweep area enable and sweep command | None | Logs starting sweep task with default areas |
| startTotalSweep | 21005 | {"mode":"totalClean"} | None | |
| stopCharging | 21012 | {"cmd":"stop"} | None | Logs stopping charging task with a UUID |
| submitClipAreaData | 21035 | {"mapId":[j],"list":[list]} | None | Logs submitting clip area data task with specified map ID and list |
| voicePacketApply | 21027 | {"cmd":"apply", "id":[str], "downUrl":[str2], "md5sum":[str3], "size":[j], "lang":[str4]} | None | |
| voicePacketDelete | 21027 | {"cmd":"delete", "id":[str]} | None | |
| voicePacketDownload | 21027 | {"cmd":"download", "id":[str], "downUrl":[str2], "md5sum":[str3], "size":[j], "lang":[str4]} | None | |
| voicePacketDownloadAndApply | 21027 | {"cmd":"downloadAndApply", "id":[str], "downUrl":[str2], "md5sum":[str3], "size":[j], "lang":[str4]} | None | |
| voicePacketGetList | 21027 | {"cmd":"getVoicePackageInfo"} | None | |
| wifiManagerGetList | 21028 | {"cmd":"getWifiList"} | 30 | |
| continueSweep | 21017 | {"cmd":"continue"} | None | Logs continuing sweep task with specified task ID |
| pauseSweep | 21017 | {"cmd":"pause"} | None | Logs pausing sweep task with specified task ID |
| requestSweepPath | 21011 | {"startPos":[j],"userId":"0","mask":0} | None | Logs requesting sweep path with specified start position |
| setSmartAreaInfo | N/A | Commands based on conditions | None | Logs setting smart area info task with specified map ID, areas, and task ID |

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

There is a UDP API on port 8790. It is normally closed, and opened when RC mode is enabled. It operates similarly to the Main API. It is not yet known if normal infoType commands can be sent here; this could allow local control. All known commands use infoType 20120

Sending movement commands: `{"infoType":21020,"data":{"ctrlCode":3013,"ctrlParams":{"speedV":0.000000,"speedW":0.000000}},"packId":27}`

Reporting current position: `{"message":"OK","infoType":21020,"x":-261,"y":-392,"angle":1410,"packId":27}`

Unknown, possibly to close connection: `{"infoType":21020,"data":{"ctrlCode":4000},"packId":194}`

# Status API

There is another API at `101.198.193.215` that appears to be used by the app for for higher-traffic uses including battery status and mapping data. As sending commmands was my primary goal this has not been explored.

Might just be a proxy for data directly from the vac, stuff that the main API doesn't care about. I would like to get status info to integrate with Home Assistant, so this API is probably next.


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



# Notes

There's a few abbreviations used in their logging: 

"jioe" is JSONIOException, "ste" is SocketTimeoutException, and 


# Android App Packages:

`com.qihoo.smarthome.sweeper.ui.o1`: RC mode, UDP reciever

