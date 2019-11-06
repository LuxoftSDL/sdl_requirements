## Policy Table Update Sequence

### **Notification on PTU request**
1.

PoliciesManager must 

notify HMI via SDL.OnStatusUpdate(UPDATE_NEEDED) on any PTU trigger

**Exception: No notification should be sent on user requested PTU from HMI (via SDL.UpdateSDL request)**

### **Policy Table Snapshot creation**
2. 
To create Policy Table Snapshot 

PoliciesManager must 

copy the Local Policy Table into memory and remove `messages` sub-section from `consumer_friendly_messages` section (See data dictionary for more details).

_Information:_  
a. The Policy Table Snapshot represents a Local Policy Table at a particular moment-in-time.  
b. `messages` sub-section is excluded from PTS with the purpose to limit the size of a request payload.

### PTS storage on a file system
3. 
	
PoliciesManager must   

store the PT snapshot as a JSON file which filename is defined in `PathToSnapshot` parameter of smartDeviceLink.ini file.

```
[Policy]
...
PathToSnapshot = sdl_snapshot.json
```

Note: filepath is defined in "SystemFilesPath"

### Sending path to Policy Table Snapshot to HMI
4. 
In case
PolicyTableUpdate is triggered

SDL must send BC.PolicyUpdate (path to SnapshotPolicyTable) to HMI for future PTS encryption and sending to backend

5.
Upon receiving BC.PolicyUpdate (SUCCESS) response from HMI

PoliciesManager must 

- change the status from `UPDATE_NEEDED` to `UPDATING` 
- and notify HMI with OnStatusUpdate(`UPDATING`)

###  Lookup the appropriate "timeout" for getting PTU
6.
Upon sending OnStatusUpdate(`UPDATING`) to HMI 

PoliciesManager must 

start timeout to wait for a response on PTU (taken from `timeout_after_x_seconds` field of LocalPT) 

### Handling HMI request for policy configuration data
7. 
In case 

HMI sends a request via SDL.GetPolicyConfigurationData

PoliciesManager must  

respond with the list of pairs URLs taken from Local PT and appropriate internal appIDs (if exist) only for the applications being now connected to SDL.

Note: In case providing the `urls` from `default` PTS section, `appID` parameter will be ommited by SDL.

### Getting urls PTS should be transfered to in case there is no application connected
8. 
To get the urls PTS should be transfered to

Policies Manager must

refer `module_config` section > `endpoints` sub-section > parent key `0x07`> key `default` in case there's no application connected at the moment.

Example of PT section
```
 "endpoints": {
        "0x07": {
          "default": [
            "http://policies.telematics.ford.com/api/policies", 
              "http://cloud.ford.com/global"
          ], 
             "123": [
            "http://policies.telematics.ford.com/api/policies", 
            "http://policies.somedomain.ford.com/api/policies", 
            "http://policies.anotherdomain.ford.com/api/policies", 
          ]
        }
      }
```

### Getting urls PTS should be transfered to (an app is registered)
9. 
To get the urls PTS should be transfered to

Policies manager must 

refer PTS `endpoints` section, key `0x07`> key `default` and key(s) app id which correspond to policyAppID(s) of the application(s) being connected to SDL now. The values must be provided as pairs (url, appID) in SDL.GetPolicyConfigurationData reponse.

Note: For the "url"(s) from "default" section appID must be set-up as "default" in SDL.GetPolicyConfigurationData response.

Example of PT
```
 "endpoints": {
        "0x07": {
          "default": [
            "http://policies.telematics.ford.com/api/policies"
          ], 
             "123": [
            "http://policies.telematics.ford.com/api/policies", 
            "http://policies.somedomain.ford.com/api/policies", 
            "http://policies.anotherdomain.ford.com/api/policies", 
          ]
        }
      }
 ```
 
 ### Sending Policy Table Snapshot from SDL to backend
 
 
