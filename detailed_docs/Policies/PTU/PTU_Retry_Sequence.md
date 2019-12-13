## Policy Table retry sequence
_(i) Info_

Policy Table Update retry sequence runs until 

a. PTU is received and successfully applied or  
b. the number of the retries is exhausted

### PROPRIETARY & HTTP

1. 
In case 

PoliciesManager does not receive the Updated PT during time defined in `timeout_after_x_seconds` section of Local PT 

SDL must

start the retry sequence

2. 
In case 

the timeout taken from `timeout_after_x_seconds` field of LocalPT or `timeout between retries` is expired before PoliciesManager receives SystemRequest with PTU from mobile application

SDL must 
- change the status to `UPDATE_NEEDED` and 
- notify HMI with OnStatusUpdate(`UPDATE_NEEDED`) 

3. 
In case 

the corresponding retry timeout expires 

SDL must 

send the new PTU request to mobile app until successful Policy Table Update has completed or  
the number of retry attempts is limited by the number of elements in `seconds_between_retries` section of LPT is exhausted

4. 

The timeout of the N retry must be count the following way:  
time for_the_ (N)retry = time_ for_ the_(N-1)retry + <"seconds_between_retries">[N] + timeout  
where N>=1 and retries are started to be numerated from "0"

In local PT:
```
local seconds_between_retries = {1, 1, 1, 1, 1}
local timeout_after_x_seconds = 30
```

Example of timeouts calculation:
```

timeout[1] = timeout_after_x_seconds + seconds_between_retries[1] -- 30 + 1 = 31
timeout[2] = timeout_after_x_seconds + seconds_between_retries[2] + timeout[2] -- 30 + 1 + 31 = 62
timeout[3] = timeout_after_x_seconds + seconds_between_retries[3] + timeout[3] -- 30 + 1 + 62 = 93
timeout[4] = timeout_after_x_seconds + seconds_between_retries[4] + timeout[4] -- 30 + 1 + 93 = 124
timeout[5] = timeout_after_x_seconds + seconds_between_retries[5] + timeout[5] -- 30 + 1 + 124 = 155
```
Note
 - These are timeouts between `OnSystemRequest` messages sent by SDL to mobile 
 - When `timeout[5]` is expired `OnSystemRequest` is not sent to mobile, but final `SDL.OnStatusUpdate(UPDATE_NEEDED)` is sent to HMI


5. 
SDL shall cycle through the list of URLs, using the next one in the list for every new policy table request over a retry sequence.  

In case of the only URL in Local Policy Table, it must always be the destination for a Policy Table Snapshot.

## Сonsequitive PTU attempts
### Initiating PTU the next ignition cycle
6. 
In case 

the Policy Table exchange is unsuccessful after the Retry Sequence is completed

SDL must 

initiate the new PT exchange sequence upon the next ignition on.

_Information_ 

Example scenarios when the system may be unable to receive a request include, but are not limited to:
- Mobile app unexpectedly crashes
- Mobile app is blocking requests (doesn't allow to set internet connection or is not able to provide internnet connection)
- Phone unexpectedly disconnects from HU
- Ignition off
- Backend server error (see s13i document links)
- Poor/no phone data connection
- Customer receives a phone call while the policy table is being downloaded into the vehicle (CDMA, Edge)

7. 
In case 

anything triggers Policy Table Update request

SDL must 

start new PTU sequence within the same ignition cycle

_See also [PTU triggers](/PTU_Triggers.md)_

8. 
In case 

the "24 hours" trigger worked, but valid PTU does not bring a certificate

SDL should 

NOT perform an additional PTU sequence for getting the PT with a certificate

### Conditions for PTU within the same IGN cycle after retry sequence failed
9. 

In case 

anything triggers Policy Table Update request

SDL must 

- start PTU sequence 

and in case of failed attempt

- restart retry sequence within the same ignition cycle


## EXTERNAL_PROPRIETARY PTU retry sequence

### PoliciesManager changes status to “UPDATE_NEEDED”
1. 
In case 

the timeout taken from `timeout_after_x_seconds` field of LocalPT is expired before PoliciesManager receives SDL.OnReceivedPolicyUpdate from HMI

SDL must  

notify HMI with OnStatusUpdate(`UPDATE_NEEDED`)   


### Sending PTS to mobile application
2. 
  
In case  
mobile application doesn't send SystemRequest with PTU
and SDL sends OnStatusUpdate(`UPDATE_NEEDED`) to HMI


**HMI is expected to:**
- start retry sequence
- set timeout
- send the new OnSystemRequest(url, appID, file) to SDL
- remove Policy Table Snapshot when retry sequence is over

3. 
In case 
SDL receives OnSystemRequest(url, appID, file) from HMI

SDL must 
- forward OnSystemRequest() to mob app
- start new `timeout_after_x_seconds timeout`
- send OnStatusUpdate(`UPDATING`) to HMI


## Diagrams

[PTU Retry Proprietary Flow](../accessories/Retry_Proprietary.png)


[PTU Retry External Proprietary Flow](../accessories/Retry_External_Proprietary.png)
