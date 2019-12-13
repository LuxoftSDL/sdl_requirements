## Policy Table Update Sequence

### **Notification on PTU request**
1.

SDL must 

notify HMI via SDL.OnStatusUpdate(UPDATE_NEEDED) on any PTU trigger

**Exception: No notification should be sent on user requested PTU from HMI (via SDL.UpdateSDL request)**

### **Policy Table Snapshot creation**
2. 
To create Policy Table Snapshot 

SDL must

copy the Local Policy Table into memory and remove `messages` sub-section from `consumer_friendly_messages` section (See data dictionary for more details).

_Information:_  
a. The Policy Table Snapshot represents a Local Policy Table at a particular moment-in-time.  
b. `messages` sub-section is excluded from PTS with the purpose to limit the size of a request payload.

### PTS storage on a file system
3. 
	
SDL must 

store the PT snapshot as a JSON file which filename is defined in `PathToSnapshot` parameter of smartDeviceLink.ini file

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

SDL must 

send BC.PolicyUpdate (path to SnapshotPolicyTable) to HMI for future PTS encryption and sending to backend

5.
Upon receiving BC.PolicyUpdate (SUCCESS) response from HMI

SDL must 

- change the status from `UPDATE_NEEDED` to `UPDATING` 
- and notify HMI with OnStatusUpdate(`UPDATING`)

###  Lookup the appropriate "timeout" for getting PTU
6.
Upon sending OnStatusUpdate(`UPDATING`) to HMI 

SDL must

start timeout to wait for a response on PTU (taken from `timeout_after_x_seconds` field of LocalPT) 

### Handling HMI request for policy configuration data
7. 
In case 

HMI sends a request via SDL.GetPolicyConfigurationData("module_config", property = "endpoints")

SDL must  

respond with the list of pairs URLs taken from Local PT and appropriate internal appIDs (if exist) only for the applications being now connected to SDL.

### Getting urls PTS should be transfered to in case there is no application connected
8. 
To get the urls PTS should be transfered to

SDL must

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

SDL must

refer PTS `endpoints` section, key `0x07`> key `default` and key(s) app id which correspond to policyAppID(s) of the application(s) being connected to SDL now. The values must be provided in SDL.GetPolicyConfigurationData(`endpoints`) response.

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
 
### Sending Policy Table Snapshot from SDL to backend/mobile application
10. 
When got from SyncP, 

SDL must 

forward OnSystemRequest(request_type=PROPRIETARY, url, timeout, appID) with encrypted PTS snapshot as a hybrid data to mobile application with `appID` value.  
`fileType` must be assigned as "JSON" in mobile app notification.

Note1: In case OnSystemRequest() is sent with "default appID" number, SDL must forward the notification with encrypted PTS snapshot as a hybrid data to connected mobile application `appID`
Note2: SDL resends the `url` parameter to mobile app via OnSystemRequest only in case it receives `url` parameter within BasicCommunication.OnSystemRequest from SyncPManager (HMI_API).
If SyncP doesn't send any URLs to SDL, it is supposed that mobile application will sent Policy Table Update data back to SDL.

HMI Note1: It's HMI responsibility to encrypt PTS file and provide it to SDL via OnSystemRequest (HMI API `fileName` parameter).

HMI Note2: It's HMI responsibility to choose an application for sending PTU and start PTU timer or retry timer after sending OnSystemRequest to SDL.

HMI Note3: HMI is responsible for initiating retry sequence. (see also [PolicyTableUpdate_Retry_Sequence]()]

HMI Note4: HMI is responsible for removing Policy Table Snapshot when retry sequence is over.

### Sending Policy Table Snapshot to backend/mobile application (got appID as "default" from HMI)
11. 

SDL must

stop the timeout started right after sending OnStatusUpdate to HMI in case SDL.OnReceivedPolicyUpdate comes from HMI

### Processing a response from a backend
12. 
Upon receiving the response from the application via SystemRequest(requestType=PROPRIETARY)  

SDL must

- read hybrid data and store it by the path specified in smartDeviceLink.ini file under "SystemFilesPath" parameter
- notify HMI with SystemRequest(requestType=PROPRIETARY, fileName, appID) about PTU has been obtained

HMI Note1: It's HMI responsibility to decode and decrypt the contents of Policy Table Update  
HMI Note2: On decoding and decrypting the PTU, HMI must_notify SDL with SDL.OnReceivedPolicyUpdate(policyfile)  
HINote3: SDL generates the name for file with stored binary data by itself and add the Integer value to each <fileName>, e.g. `<1fileName>`(applicable for IVSU and PROPRIETARY requestTypes)
 
#### PTU Validation
13. 
After getting OnReceivedPolicyUpdate (`policyFile`) from HMI

SDL must
- stop timeout 
- validate the Policy Table Update (`policyFile`) according to Data Dictionary statuses of optional, required, or omitted:

1) Validation must reject a policy table update if it include fields with a status of ‘omitted’
2) Validation must reject a policy table update if it does not include fields with a status of ‘required’

Note: In case section with required status "optional/omitted" is ommited in Updated PT, and a field of this section is marked as required, the validation of the mentioned field is not "required" (i.e. policy table must be considered as valid).

14. 
Right after successful validation of received PTU

SDL must

change the status to UP_TO_DATE and notify HMI with OnStatusUpdate(UP_TO_DATE)

15. 
In case PTU validation fails

SDL must

- log the error locally
- discard the Policy Table Update with No notification of Cloud about invalid data
- notify HMI with OnStatusUpdate(UPDATE_NEEDED)

#### PTU merge
#### PTU merge into Local Policy Table
16. 

In case of successful PTU validation   

SDL must 

replace the following sections of the Local Policy Table with the corresponding sections from PTU:
* `module_config`,
* `functional_groupings`,
* `app_policies`

17. 

In case 

the `consumer_friendly_messages` section of PTU contains a `messages` subsection  

SDL must

replace the `consumer_friendly_messages` portion of the Local Policy Table with the same section from PTU

18. 
In case 

the Updated PT omits `consumer_friendly_messages` section  

SDL must

maintain the current `consumer_friendly_messages` section in Local PT.

#### PTU file removal on PTU sequence end
19. 
	
Policies Manager must delete the file with Policy Table Update (got by SDL.OnReceivedPolicyUpdate) for the both cases:

1) After successful merge Policy Table Update into Local Policy Table
or
2) Validation failure against Data Dictionary

#### DecryptCertificate
20. 
In case 

SDL gets an Updated PT with non-empty "certificate" field  

SDL must 

- copy the value from "certificate" field to the file named "certificate" and
- store it in the folder defined by existing `AppStorageFolder` param

21. 
In case 

SDL stores the `certificate` file

SDL must 

send BC.DecryptCertificate_request with the path to `certificate` file to HMI

22. 
In case 

SDL receives successful BC.DecryptCertificate_response from HMI  

SDL must 
copy the decrypted certificate from the file to the "certificate" field of the policy database

23. 
In case 

SDL has copied the decrypted certificate from the file to the "certificate" field of the policy database 

SDL must 

remove the file

## Diagrams

![PTU_EXTERNAL_PROPRIETARY](../accessories/PTU_EXTERNAL_PROPRIETARY.png)