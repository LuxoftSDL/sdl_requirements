## PROPRIETARY Policy Table Update Sequence

##  Sending Policy Table Snapshot from SDL to backend

### **Notification on PTU request**
1.

PoliciesManager must 

notify HMI via SDL.OnStatusUpdate(UPDATE_NEEDED) on any PTU trigger

EXTERNAL_PROPRIETARY exception: No notification should be sent on user requested PTU from HMI (via SDL.UpdateSDL request).

_Note: the source of the PolicyTableUpdate is the Policies Cloud._


2. 	

To create Policy Table Snapshot 

PoliciesManager must 

copy the Local Policy Table into memory and remove `messages` sub-section from `consumer_friendly_messages` section (See data dictionary for more details).

_Information:_  
a. The Policy Table Snapshot represents a Local Policy Table at a particular moment-in-time.  
b. `messages` sub-section is excluded from PTS with the purpose to limit the size of a request payload.

### Sending path to Policy Table Snapshot to HMI
3. 	

In case

SDL is built with "DEXTENDED_POLICY: ON" "-DEXTENDED_POLICY: PROPRIETARY" flag or without this flaf at all
and PolicyTableUpdate is triggered 

SDL must
send BC.PolicyUpdate (`path to SnapshotPolicyTable`, `timeout from policies`, `set of retry timeouts`) to HMI

4. 

Upon receiving BC.PolicyUpdate (SUCCESS) response from HMI

PoliciesManager must 

- change the status from `UPDATE_NEEDED` to `UPDATING` 
- and notify HMI with OnStatusUpdate(`UPDATING`)  


###  Lookup the appropriate "timeout" for getting PTU
5.
Upon sending OnStatusUpdate(`UPDATING`) to HMI 

PoliciesManager must 

start timeout to wait for a response on PTU (taken from `timeout_after_x_seconds` field of LocalPT) 

6. 

To define the timeout to wait for a response on PTU

Policies manager must 

refer PTS `module_config` section, key `timeout_after_x_seconds`

Example of PT:
```
 "module_config": {
      "preloaded_pt": true,
      "vehicle_make": "",
      "vehicle_model": "",
      "vehicle_year": "",
      "exchange_after_x_ignition_cycles": 100,
      "exchange_after_x_kilometers": 1800,
      "exchange_after_x_days": 30,
      "timeout_after_x_seconds": 60,
      "seconds_between_retries": [
        1,
        5,
        25,
        125,
        625
      ],
```

### Handling HMI request for policy configuration data
7. 

In case

SDL is built with "DEXTENDED_POLICY: ON" "-DEXTENDED_POLICY: PROPRIETARY" flag or without this flaf at all

and SDL gets SDL.GetPolicyConfigurationData (service: 7) from HMI

SDL must

respond SDL.GetPolicyConfigurationData_response (SUCCESS, urls: array(`SDL-chosen appID` + `url from policy database for service 7`)) to HMI

_Information_
Related policies section:
```
      "endpoints": {
        "0x07": {
          "default": [
            "http://policies.url.com"
          ]
        }
      },
```

### Getting `urls` PTS should be transfered to
8. 

To get the `urls` PTS should be transfered to 

Policies Manager must 

refer PTS `endpoints` section, key "0x07" for the appropriate `<app id>` which was chosen for PTS transferring

Example of PT:
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
### HMI chooses to process PTU via mob app

9.

In case

SDL is built with "DEXTENDED_POLICY: ON" "-DEXTENDED_POLICY: PROPRIETARY" flag or without this flag at all

and HMI sends BC.OnSystemRequest (PROPRIETARY, fileName: `<path to Snapshot`>, url, appID)

SDL must

send OnSystemRequest to the specified `appID` app with Snapshot and Binary Header (below) in payload

```
 {
	"HTTPRequest": {
		"headers": {
			"ContentType": "application/json",
			"ConnectTimeout": <int value from policy table:timeout_after_x_seconds>,
			"DoOutput": true,
			"DoInput": true,
			"UseCaches": false,
			"RequestMethod": "POST",
			"ReadTimeout": <int value from policy table:timeout_after_x_seconds>,
			"InstanceFollowRedirects": false,
			"charset": "utf-8",
			"Content_Length": <value of the length of the "body" string>
		},
		"body": "{"data":["this is either the syncp packet or raw policy table BgcYTArvdz62FO7yHgOTXtE3EP0E0WsbFeDqzYY="]}"
	}
}
```

_Information_

a. OnSystemRequest with SnapshotPT (= binary data) should be sent over "Bulk" Service (15) to mobile app


10. 

Policies Manager must 

randomly select the application through which to send the Policy Table packet
and request an update to its Local Policy Table only through apps with HMI status of BACKGROUND, LIMITED, and FULL.


If there are no mobile apps with any of these statuses, the system must use an app with an HMI Level of NONE.


11. 

PoliciesManager must 

stop the timeout started right after sending OnStatusUpdate to HMI
in case SDL.OnReceivedPolicyUpdate comes from HMI


### Processing a response from a backend

12. 22484
	
In case  
SDL is built with "-DEXTENDED_POLICY: PROPRIETARY" flag or without this flag at all  
and SDL has sent OnSystemRequest with SnapshotPT to mobile app  
and SDL gets SystemRequest with UpdatedPT in payload during `timeout_after_x_seconds` value taken from Policy Database and started after OnSystemRequest sending out to mobile app

SDL must

send BasicCommunication.SystemRequest (`<path to UpdatedPT>`, PROPRIETARY, params) to HMI

_Information_

a. Path for SDL to store the UpdatedPT is defined by `SystemFilesPath` (value defined in [SmartDeviceLink.ini. file]((https://github.com/smartdevicelink/sdl_core/blob/develop/src/appMain/smartDeviceLink.ini)))  
b. UpdatedPT is not applied by SDL until OnReceivedPolicyUpdate notification from HMI  
c. SDL should expect SystemRequest with UpdatedPT (= binary data) over "Bulk" Service (15) from mobile app  
d. If SystemRequest is not received during `timeout_after_x_seconds`, SDL should start the "Retry Sequence" 

13. 22485

In case  
SDL is built with "-DEXTENDED_POLICY: PROPRIETARY" flag or without this flag at all  
and SDL has sent BasicCommunication.SystemRequest to HMI  
and SDL gets BasicCommunication.SystemRequest_response (`<resultCode>`) during "SDL timeout for HMI requests" (= 10 sec by default) from HMI

SDL must
send SystemRequest_response (`<resultCode from HMI response>`) to mobile app

_Information_  
a. If SDL's timeout is expired with no response from HMI, SDL responds with GENERIC_ERROR to mobile app

14. 22486
In case  
SDL is built with "-DEXTENDED_POLICY: PROPRIETARY" flag or without this flag at all  
and SDL gets SDL.OnReceivedPolicyUpdate (`<path to UpdatedPT>`) from HMI

SDL must  
apply the valid UpdatedPT to Policy Database

#### PTU Validation

15. 18184

After Base-64 decoding

SDL must 
validate the Policy Table Update against Policy_Table_Data_Dictionar.xlsx statuses of optional, required, or omitted:

1) Validation must reject a policy table update if it include fields with a status of ‘omitted’
2) Validation must reject a policy table update if it does not include fields with a status of ‘required’

16. 18803

Right after successful validation of received PTU

PoliciesManager must 

change the status to `UP_TO_DATE` and notify HMI with OnStatusUpdate(`UP_TO_DATE`)

17. 18187

In case PTU validation fails

SDL must 
- log the error locally
- discard the Policy Table Update with No notification of Cloud about invalid data
- notify HMI with OnStatusUpdate(UPDATE_NEEDED)

#### PTU merge

18. 18190

In case of successful PTU validation   

SDL must 

replace the following sections of the Local Policy Table with the corresponding sections from PTU:
* `module_config`,
* `functional_groupings`,
* `app_policies`

19. 18192

In case 

the `consumer_friendly_messages` section of PTU contains a `messages` subsection  

SDL must

replace the `consumer_friendly_messages` portion of the Local Policy Table with the same section from PTU

Note: Refer Data Dictionary for Policy Table structure information

## Diagrams

### PROPRIETARY - PTU using mob app  
|||
![PTU_Proprietary](./PTU_PROPRIETARY.png)
|||

### PROPRIETARY - PTU using in-vehicle modem 

|||
![PTU_Proprietary_modem_success](./PTU_PROPRIETARY_modem_success.png)
|||

|||
![PTU_Proprietary_modem_fail](./PTU_PROPRIETARY_modem_failure.png)
|||
