# [RHEL] Subscription-manager command fails with "NetworkException: Network error code: 400"

Owner: Nam Tran
Last edited time: September 9, 2025 10:29 AM

### **Solution Verified** Updated

## Environment

- Red Hat Enterprise Linux
- Red Hat Subscription Management (RHSM)
- Simple Content Access

## Issue

- Subscription-manager command fails with `Network error code: 400`
    
    [**Raw**](https://access.redhat.com/solutions/5187461#)
    
    ```
    [user@system ~]# subscription-manager list --available --all
    Error updating system data on the server, see /var/log/rhsm/rhsm.log for more details.
    [ERROR] rhsmcertd-worker:21003:MainThread @rhsmcertd-worker:148 -
    Error while updating certificates using daemon
    [ERROR] rhsmcertd-worker:21003:MainThread @rhsmcertd-worker:150 - Network error code: 400
    
    ```
    
- Yum commands fails with error `Network error code: 400`
    
    [**Raw**](https://access.redhat.com/solutions/5187461#)
    
    ```
    # yum repolist
    Loaded plugins:  product-id, security, subscription-manager
    Network error code: 400
    repolist: 0
    
    ```
    
- Subscription-manager command fails with `HTTP error code 400: Bad Request`
    
    [**Raw**](https://access.redhat.com/solutions/5187461#)
    
    ```
    [user@system ~]subscription-manager refresh
    Unknown server reply (HTTP error code 400: Bad Request):
    <html><13>
    <head><title>400 The SSL certificate error</title></head><13>
    <body><13>
    <center><h1>400 Bad Request</h1></center><13>
    <center>The SSL certificate error</center><13>
    <hr><center>openresty</center><13>
    </body><13>
    </html><13>
    
    ```
    
- Below errors in `/var/log/rhsm/rhsm.log`.
    
    [**Raw**](https://access.redhat.com/solutions/5187461#)
    
    ```
    rhsm.connection.UnknownContentException: Unknown content error (HTTP 400 - Bad Request, type text/html, len 212)
    2024-10-09 11:55:34,181 [ERROR] rhsmcertd-worker:130542:MainThread @rhsmcertd_worker.py:281 - Error while updating certificates using daemon
    2024-10-09 11:55:34,181 [ERROR] rhsmcertd-worker:130542:MainThread @rhsmcertd_worker.py:283 - [Errno 13] Permission denied
    Traceback (most recent call last):
    
    ```
    

## Resolution

- To delete duplicate/old system profile entries which were present in Red Hat Customer Portal, use API by referring to [How to delete the system profile from the Red Hat Customer Portal using RHSM APIs ?](https://access.redhat.com/solutions/7070895).
    
    If this does not resolve the issue, proceed to the next step.
    
- Check the directory list under `/etc/pki/product/` for unusual/incorrect directory structure like:
    
    [**Raw**](https://access.redhat.com/solutions/5187461#)
    
    ```
    # tree /etc/pki/product/
    /etc/pki/product/
    └── product
     └── product
         └── product
             └── product
                 └── product
                     └── product
                         └── product
    
    7 directories, 0 files
    
    ```
    
    To fix the structure, delete extra subdirectories from under `/etc/pki/product/` location.
    
    [**Raw**](https://access.redhat.com/solutions/5187461#)
    
    ```
    # cd /etc/pki/product/
    # rm -rf product/
    
    ```
    
    Copy respective `.pem` file from `/etc/pki/product-default/` directory to `/etc/pki/product/` directory ( **just copy, do not delete /etc/pki/product-default/*.pem** ).
    
    [**Raw**](https://access.redhat.com/solutions/5187461#)
    
    ```
    # cp /etc/pki/product-default/479.pem /etc/pki/product/
    
    ```
    
    Verify the structure which should look like:
    
    [**Raw**](https://access.redhat.com/solutions/5187461#)
    
    ```
    # tree /etc/pki/product
    /etc/pki/product
    └── 69.pem
    
    0 directories, 1 files
    
    ```
    
- Clean and remove old certificate files and corrupted subscripiton-manager data. Then re-register the system freshly and subscribe.
    
    [**Raw**](https://access.redhat.com/solutions/5187461#)
    
    ```
    # rm -rf /etc/pki/consumer
    # rm -rf /etc/pki/entitlement
    # subscription-manager clean
    # subscription-manager register
    
    ```
    

## Root Cause

- Incorrect directory structure for installed products.
- Duplicate entries in customer portal for same system's hostname.
- Corrupted identity certificates present in `/etc/pki/consumer/` location causing the failure of registration or subscription-manager commands.

## Diagnostic Steps

- Traceback found in `/var/log/rhsm/rhsm.log` file:

[**Raw**](https://access.redhat.com/solutions/5187461#)

```
2020-07-16 18:34:51,705 [DEBUG] subscription-manager:5237 @connection.py:589 - Response: status=400
2020-07-16 18:34:51,706 [ERROR] subscription-manager:5237 @connection.py:618 - Response: 400
2020-07-16 18:34:51,706 [ERROR] subscription-manager:5237 @connection.py:619 - JSON parsing error: No JSON object could be decoded
2020-07-16 18:34:51,706 [ERROR] subscription-manager:5237 @managercli.py:174 - Unable to perform refresh due to the following exception: Network error code: 400
2020-07-16 18:34:51,706 [ERROR] subscription-manager:5237 @managercli.py:175 - Network error code: 400
Traceback (most recent call last):
  File "/usr/share/rhsm/subscription_manager/managercli.py", line 622, in _do_command
    self.entcertlib.update()
  File "/usr/share/rhsm/subscription_manager/certlib.py", line 31, in update
    self.report = self.locker.run(self._do_update)
  File "/usr/share/rhsm/subscription_manager/certlib.py", line 17, in run
    return action()
  File "/usr/share/rhsm/subscription_manager/entcertlib.py", line 43, in _do_update
    return action.perform()
  File "/usr/share/rhsm/subscription_manager/entcertlib.py", line 119, in perform
    expected = self._get_expected_serials()
  File "/usr/share/rhsm/subscription_manager/entcertlib.py", line 254, in _get_expected_serials
    exp = self.get_certificate_serials_list()
  File "/usr/share/rhsm/subscription_manager/entcertlib.py", line 234, in get_certificate_serials_list
    reply = self.uep.getCertificateSerials(identity.uuid)
  File "/usr/lib64/python2.6/site-packages/rhsm/connection.py", line 1145, in getCertificateSerials
    return self.conn.request_get(method)
  File "/usr/lib64/python2.6/site-packages/rhsm/connection.py", line 681, in request_get
    return self._request("GET", method)
  File "/usr/lib64/python2.6/site-packages/rhsm/connection.py", line 598, in _request
    self.validateResponse(result, request_type, handler)
  File "/usr/lib64/python2.6/site-packages/rhsm/connection.py", line 668, in validateResponse
    raise NetworkException(response['status'])
NetworkException: Network error code: 400

```

Or:

[**Raw**](https://access.redhat.com/solutions/5187461#)

```
  File "/usr/lib64/python2.7/site-packages/rhsm/connection.py", line 713, in validateResponse
    raise NetworkException(response['status'])
NetworkException: HTTP error (400 - Bad Request)

```

Or:

[**Raw**](https://access.redhat.com/solutions/5187461#)

```
  File "/usr/lib64/python3.6/socket.py", line 713, in create_connection
    sock.connect(sa)
PermissionError: [Errno 13] Permission denied

```

Or:

[**Raw**](https://access.redhat.com/solutions/5187461#)

```
2024-12-30 11:11:46,387 [ERROR] yum:409310:MainThread @connection.py:898 - Response: 400
2024-12-30 11:11:46,387 [ERROR] yum:409310:MainThread @connection.py:899 - JSON parsing error: Expecting value: line 1 column 1 (char 0)
2024-12-30 11:11:46,387 [ERROR] yum:409310:MainThread @repolib.py:362 - UnknownContentException: Unknown content error (HTTP 400 - Bad Request, type text/html, len 212)
```