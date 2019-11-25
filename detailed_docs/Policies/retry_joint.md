## Policy Table retry sequence
_(i) Info_

Policy Table Update retry sequence runs until 

a. PTU is received and successfully applied or  
b. the number of the retries is expired

### PROPRIETARY & HTTP

1. 
In case 

PoliciesManager does not receive the Updated PT during time defined in `timeout_after_x_seconds` section of Local PT 

PoliciesManager must

start the retry sequence

2. 
In case 

the timeout taken from `timeout_after_x_seconds` field of LocalPT or `timeout between retries` is expired before PoliciesManager receives SystemRequest with PTU from mobile application

PoliciesManager must 
- change the status to `UPDATE_NEEDED` and 
- notify HMI with OnStatusUpdate(`UPDATE_NEEDED`) 

3. 
In case 

the corresponding retry timeout expires 

PoliciesManager must 

send the new PTU request to mobile app until successful Policy Table Update has completed or the number of retry attempts is limited by the number of elements in `seconds_between_retries` section of LPT

4. 

The timeout of the N retry must be count the following way:  
time for_the_ (N)retry = time_ for_ the_(N-1)retry + <"seconds_between_retries">[N] + timeout  
where N>=1 and retries are started to be numerated from "0"

_Example:_

RetryTimeout_1 = <seconds_between_retries>[1] + timeout_after_x_seconds;
RetryTimeout_2 = RetryTimeout_1 + <seconds_between_retries>[2] + timeout
RetryTimeout_3 = RetryTimeout_2 + <seconds_between_retries>[3] + timeout
..
etc

Example of the Local Policy Table "seconds_between_retries" section

```
 "seconds_between_retries": [
        1,
        5,
        25,
        125,
        625
      ],
```

5. 
The PoliciesManager shall cycle through the list of URLs, using the next one in the list for every new policy table request over a retry sequence.  

In case of the only URL in Local Policy Table, it must always be the destination for a Policy Table Snapshot.

## Сonsequitive PTU attempts
6. 
In case 

the policy table exchange is unsuccessful after the retry strategy is completed

Policy Manager must 

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

Policy Manager must 

restart retry sequence within the same ignition cycle

_See also [PTU triggers]()_

8. 
In case 

the "24 hours" trigger worked, but valid PTU does not bring a certificate

SDL should 

NOT perform a retry sequence for getting the PTU with a certificate.


## EXTERNAL_PROPRIETARY PTU retry sequence

### PoliciesManager changes status to “UPDATE_NEEDED”
1. 

PoliciesManager must  

change the status to `UPDATE_NEEDED` and  
notify HMI with OnStatusUpdate(`UPDATE_NEEDED`)   
in case the timeout taken from `timeout_after_x_seconds` field of LocalPT or `timeout between retries` is expired before PoliciesManager receives SDL.OnReceivedPolicyUpdate from HMI.

### Sending PTS to mobile application
2. 
  
In case the corresponding PTU timeout or retry timeout expires,  

HMI must  
send the new OnSystemRequest to SDL until successful Policy Table Update has finished or the number of retry attempts is limited by the number of elements in `seconds_between_retries` section of LPT.

The timeout of the N retry must be count the following way by HMI:
On getting SDL.PolicyUpdate(retry[],timeout) to store retry[] values.

Example:
```
0_RetryTimeout = retry[0] + timeout_after_x_seconds;
1st_RetryTimeout = 0_RetryTimeout + retry[1] + timeout
2nd_RetryTimeout =1st_RetryTimeout + retry[2] +timeout
..
etc
```
HMI Note: HMI is responsible for removing Policy Table Snapshot when retry sequence is over.

### Initiating PTU the next ignition cycle 
3. 

In case 

the policy table exchange is unsuccessful after the retry strategy is completed

Policy Manager must 

initiate the new PT exchange sequence upon the next ignition on.

_Information_ 

Example scenarios when the system may be unable to receive a request include, but are not limited to:
- Mobile app unexpectedly crashes
- Mobile app is blocking requests (doesn't allow to set internet connection or is not able to provide internnet connection)
- Phone unexpectedly disconnects from HU
- Ignition off
- Backend server error
- Poor/no phone data connection
- Customer receives a phone call while the policy table is being downloaded into the vehicle (CDMA, Edge)

### Conditions for PTU within the same IGN cycle after retry sequence failed
4. 

In case 

anything triggers Policy Table Update request

Policy Manager must 

start PTU sequence and in case of failed attempt
restart retry sequence within the same ignition cycle

## Diagrams
|||
[PM starts and stops timeout](accessories/PM_starts_and_stops_timeout.jpg)
|||
|||
[Policy Table retry sequence](accessories/policies_retry_sequence.png)