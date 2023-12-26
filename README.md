# Authentication

A QID and SID, and vacuum SN are required. (User and session IDs)

## Obtaining the QID:

1. Sign into the 360 website using this link: [https://i.360.cn/reg/?src=pcw_open_app&destUrl=http%3A%2F%2Fdev.app.360.cn%2Fapp%2Flist](https://i.360.cn/login/?src=pcw_home&destUrl=https://www.360.cn/)
2. Visit [http://open.app.360.cn/developer/](http://open.app.360.cn/bbs/index) and copy the ten digit number beginning in `360Uxxxxxxxx` at the top right. This is your QID. It should look like this: `3123795435`

## Obtaining the SID:

The SID is a 32-character long session ID used to authenticate HTTP requests to the API. Obtaining this key is currently non-trivial and will be documented after a better method is found for extracting it. 

## Obtaining the SN:

The SN can be viewed in Toolbox->Settings->Device Info on the app. 

## Generating a cookie

Once you have your SID and UID, you can use the following cookie to authenticate:

```text
q=u=&t=1;t=&v=2.0&a=1; qid=1234123412; sid=4a20145c62428d31b52b53c9ccbfcee4
```



# Sending commands

Most commands are send to the `clean/cmd/send` endpoint. Here is an example of the "Find Robot" command, which makes it speak. 

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

# Command List



# API endpoints:

This is a list of endpoints which is used by the 360 mobile app to control 360 smart ai devices like 360 S5 / S6 / S7 vacuum cleaner.


# clean
https://q.smart.360.cn/clean/cmd/send

Most of the commands are sent here. Includes heartbeat, sweep, voice, notify, etc.

https://q.smart.360.cn/clean/devuser/updateInfo

https://q.smart.360.cn/clean/record/statis

Returns total lifetime cleaning statistics for a given serial number, and the human readable string (data.clues) used on the Cleaning Report page of the app. 

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





# map
https://q.smart.360.cn/clean/record/getBackupMapList
https://q.smart.360.cn/s3_file_new/iot-master-clean-online-pub1-bjyt

# json
https://smart.360.cn/clean/errorInfo_us.json
https://smart.360.cn/clean/share_act.json
https://smart.360.cn/clean_dev/list_en_us.json

# sdk
https://sdk.s.360.cn/ak/e995f98d56967d946471af29d7bf99f1.html?m2=8D7ACEB0-8EA1-4AD5-9339-34D01918FC4A

# update
https://p.s.360.cn/update/update.php

# upgrade
https://ota3.jia.360.cn/upgrade/getNewVersion

# statistics
https://p.s.360.cn/pstat/plog.php

# common
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

# product images
http://p0.qhimg.com/t0123a732c50f157afd.png
