## HTTP Policy Table Update Sequence

###  Sending Policy Table Snapshot from SDL to backend

### **Notification on PTU request**

1. 19072
PoliciesManager must 

notify HMI via SDL.OnStatusUpdate(UPDATE_NEEDED) on any PTU trigger


_Note: the source of the PolicyTableUpdate is the Policies Cloud._

### **Policy Table Snapshot creation**

2. 18053 
To create Policy Table Snapshot 

PoliciesManager must 

copy the Local Policy Table into memory and remove `messages` sub-section from `consumer_friendly_messages` section (See data dictionary for more details).

_Information:_  
a. The Policy Table Snapshot represents a Local Policy Table at a particular moment-in-time.  
b. `messages` sub-section is excluded from PTS with the purpose to limit the size of a request payload.

### **Select an application PTS will be transfered with**

3. 18272
Policies Manager must 

randomly select the application through which to send the Policy Table packet
and request an update to its Local Policy Table only through apps with HMI status of BACKGROUND, LIMITED, and FULL.


If there are no mobile apps with any of these statuses, the system must use an app with an HMI Level of NONE.

### **Define the URL(s) Policy Table Snapshot will be transfered to**

4. 18106
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

### **Lookup the appropriate "timeout" for getting PTU**

5. 18114

To define the timeout to wait for a response on PTU

Policies manager must 

refer PTS `module_config` section, key <timeout_after_x_seconds>

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

### **Sending Policy Table Snapshot to backend/mobile application**

6. 18176
SDL must 

send PTS snapshot as a binary data via OnSystemRequest mobile API from the system to backend. 

The `url` PTS will be forwarded to and `timeout` must be taken from the Local Policy Table.

Note: If no `url` is provided in Local Policy Table, it is supposed that mobile application will sent Policy Table Update data back to SDL.

### **Start "timeout" countdown for getting an answer with Policy Table Update**

7. 18179
PoliciesManager must 

start timeout taken from `timeout_after_x_seconds` field of LocalPT right after OnSystemRequest is sent out to mobile app.

### **Start "timeout" countdown**

8. nnn
PoliciesManager must 

stop timeout started right after OnSystemRequest is sent out to mobile app in case SDL.OnReceivedPolicyUpdate comes from HMI.

### Processing an answer from a backend
### **Getting Policy Table Update on SDL**

9. 
Upon receiving the response (before timeout) from the application via SystemRequest  

SDL must 

stop the timer of PTU "timeout" and Base64-decode the payload, which is the Policy Table Update.

### **SDL performs the validation of Policy Table Update**

10. 18184
After Base-64 decoding

SDL must 
validate the Policy Table Update against Policy_Table_Data_Dictionar.xlsx statuses of optional, required, or omitted:

1) Validation must reject a policy table update if it include fields with a status of ‘omitted’
2) Validation must reject a policy table update if it does not include fields with a status of ‘required’

**Policy Table Update validation failure exception**
11. 18187
In case PTU validation fails

SDL must 
- log the error locally
- discard the Policy Table Update with No notification of Cloud about invalid data
- notify HMI with OnStatusUpdate(UPDATE_NEEDED)

**Change PolicyUpdate status to UP_TO_DATE**
12. 18803
Right after successful validation of received PTU

PoliciesManager must 

change the status to `UP_TO_DATE` and notify HMI with OnStatusUpdate(`UP_TO_DATE`)

### **Merge of Policy Table Update into Local Policy Table**

13. 18190
In case of successful PTU validation   

SDL must 

replace the following sections of the Local Policy Table with the corresponding sections from PTU:
* `module_config`,
* `functional_groupings`,
* `app_policies`

### **Merge of Policy Table Update into Local Policy Table ("consumer_friendly_messages")**

14. 18192

In case 

the `consumer_friendly_messages` section of PTU contains a `messages` subsection  

SDL must

replace the `consumer_friendly_messages` portion of the Local Policy Table with the same section from PTU

Note: Refer Data Dictionary for Policy Table structure information

15. 22734

In case the Updated PT omits `consumer_friendly_messages` section, 

PoliciesManager must 

maintain the current `consumer_friendly_messages` section in Local PT.

## Diagrams

### PROPRIETARY - PTU using mob app  
|||
![PTU_HTTP](./PTU_HTTP.png)
|||
