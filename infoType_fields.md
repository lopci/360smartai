Here is a full list of untested infofields:

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
