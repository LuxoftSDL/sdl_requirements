## Policy Table retry sequence

### PoliciesManager changes status to “UPDATE_NEEDED”
1. 

PoliciesManager must  

change the status to `UPDATE_NEEDED` and  
notify HMI with OnStatusUpdate(`UPDATE_NEEDED`)   
in case the timeout taken from `timeout_after_x_seconds` field of LocalPT or `timeout between retries` is expired before PoliciesManager receives SDL.OnReceivedPolicyUpdate from HMI.

### Sending PTS to mobile application
2. 

When got from SyncP, 

SDL must forward OnSystemRequest(request_type=PROPRIETARY, url, appID) with encrypted PTS snapshot as a hybrid data to mobile application with `appID` value.  
`fileType` must be assigned as "JSON" in mobile app notification.

Note: SDL resends the `url` parameter to mobile app via OnSystemRequest only in case it receives `url` parameter within BasicCommunication.OnSystemRequest from SyncPManager (HMI_API).
If SyncP doesn't send any URLs to SDL, it is supposed that mobile application will sent Policy Table Update data back to SDL.

SyncP Note1: It's HMI responsibility to encrypt PTS file and provide it to SDL via OnSystemRequest (HMI API `fileName` parameter).

SyncP Note2: It's HMI responsibility to choose an application for sending PTU and start PTU timer or retry timer after sending OnSystemRequest to SDL.

SyncP Note3: HMI is responsible for initiating retry sequence.  
In case the corresponding PTU timout or retry timeout expires,  

HMI must  
send the new OnSystemRequest to SDL until successful Policy Table Update has finished or the number of retry attempts is limited by the number of elements in `seconds_between_retries` section of LPT.

The timeout of the N retry must be count the following way by SyncP:
1) On getting SDL.PolicyUpdate(retry[],timeout) to store retry[] values.

Example:
```
0_RetryTimeout = retry[0] + timeout;
1st_RetryTimeout = 0_RetryTimeout + retry[1] + timeout
2nd_RetryTimeout =1st_RetryTimeout + retry[2] +timeout
..
etc
```
SyncP Note4: HMI is responsible for removing Policy Table Snapshot when retry sequence is over.

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

Note: the system shall respond to error codes provided by the backend server as described in the S13i specification (2.5.4.2 Status Code).

### Conditions for PTU within the same IGN cycle after retry sequence failed
4. 

In case 

anything triggers Policy Table Update request

Policy Manager must 

restart retry sequence within the same ignition cycle

###  SDL must NOT perform retry sequence in case PTU does not bring the valid certificate
5. 

In case 

the "24 hours" trigger worked, but valid PTU does not bring a certificate

SDL should 

NOT perform a retry sequence for getting the PTU with a certificate.

_Information_  
a. The triggers for checking the cert expiration status are:  
-> ignition on  
-> TLS handshake  
b. in case module's certificate in policies is expired or invalid, the TLS handshake will fail  
c. in case TLS handshake fails because module's certificate is expired or invalid, the "count_of_TLS_errors" must not be incremented  
d. current model of Ford's backend implementation in Opensource: certificate is always present in PTU (meaning, each successfull request for policies update would bring a certificate in updated PT)
