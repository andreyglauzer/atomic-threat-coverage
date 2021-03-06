| Title                    | Interactive Logon to Server Systems       |
|:-------------------------|:------------------|
| **Description**          | Detects interactive console logons to |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0008: Lateral Movement](https://attack.mitre.org/tactics/TA0008)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1078: Valid Accounts](https://attack.mitre.org/techniques/T1078)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0004_4624_windows_account_logon](../Data_Needed/DN_0004_4624_windows_account_logon.md)</li><li>[DN_0040_528_user_successfully_logged_on_to_a_computer](../Data_Needed/DN_0040_528_user_successfully_logged_on_to_a_computer.md)</li><li>[DN_0041_529_logon_failure](../Data_Needed/DN_0041_529_logon_failure.md)</li><li>[DN_0057_4625_account_failed_to_logon](../Data_Needed/DN_0057_4625_account_failed_to_logon.md)</li></ul>  |
| **Trigger**              |  There is no documented Trigger for this Detection Rule yet  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>Administrative activity via KVM or ILO board</li></ul>  |
| **Development Status**   |  Development Status wasn't defined for this Detection Rule yet  |
| **References**           |  There are no documented References for this Detection Rule yet  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: Interactive Logon to Server Systems
id: 3ff152b2-1388-4984-9cd9-a323323fdadf
description: Detects interactive console logons to
author: Florian Roth
date: 2017/03/17
tags:
    - attack.lateral_movement
    - attack.t1078
logsource:
    product: windows
    service: security
detection:
    selection:
        EventID:
            - 528
            - 529
            - 4624
            - 4625
        LogonType: 2
        ComputerName:
            - '%ServerSystems%'
            - '%DomainControllers%'
    filter:
        LogonProcessName: Advapi
        ComputerName: '%Workstations%'
    condition: selection and not filter
falsepositives:
    - Administrative activity via KVM or ILO board
level: medium

```





### powershell
    
```
Get-WinEvent -LogName Security | where {((($_.ID -eq "528" -or $_.ID -eq "529" -or $_.ID -eq "4624" -or $_.ID -eq "4625") -and $_.message -match "LogonType.*2" -and ($_.message -match "%ServerSystems%" -or $_.message -match "%DomainControllers%")) -and  -not ($_.message -match "LogonProcessName.*Advapi" -and $_.message -match "ComputerName.*%Workstations%")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Security" AND (winlog.event_id:("528" OR "529" OR "4624" OR "4625") AND winlog.event_data.LogonType:"2" AND winlog.computer_name:("%ServerSystems%" OR "%DomainControllers%")) AND (NOT (winlog.event_data.LogonProcessName:"Advapi" AND winlog.computer_name:"%Workstations%")))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/3ff152b2-1388-4984-9cd9-a323323fdadf <<EOF\n{\n  "metadata": {\n    "title": "Interactive Logon to Server Systems",\n    "description": "Detects interactive console logons to",\n    "tags": [\n      "attack.lateral_movement",\n      "attack.t1078"\n    ],\n    "query": "(winlog.channel:\\"Security\\" AND (winlog.event_id:(\\"528\\" OR \\"529\\" OR \\"4624\\" OR \\"4625\\") AND winlog.event_data.LogonType:\\"2\\" AND winlog.computer_name:(\\"%ServerSystems%\\" OR \\"%DomainControllers%\\")) AND (NOT (winlog.event_data.LogonProcessName:\\"Advapi\\" AND winlog.computer_name:\\"%Workstations%\\")))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.channel:\\"Security\\" AND (winlog.event_id:(\\"528\\" OR \\"529\\" OR \\"4624\\" OR \\"4625\\") AND winlog.event_data.LogonType:\\"2\\" AND winlog.computer_name:(\\"%ServerSystems%\\" OR \\"%DomainControllers%\\")) AND (NOT (winlog.event_data.LogonProcessName:\\"Advapi\\" AND winlog.computer_name:\\"%Workstations%\\")))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "email": {\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Interactive Logon to Server Systems\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
((EventID:("528" "529" "4624" "4625") AND LogonType:"2" AND ComputerName:("%ServerSystems%" "%DomainControllers%")) AND (NOT (LogonProcessName:"Advapi" AND ComputerName:"%Workstations%")))
```


### splunk
    
```
(source="WinEventLog:Security" ((EventCode="528" OR EventCode="529" OR EventCode="4624" OR EventCode="4625") LogonType="2" (ComputerName="%ServerSystems%" OR ComputerName="%DomainControllers%")) NOT (LogonProcessName="Advapi" ComputerName="%Workstations%"))
```


### logpoint
    
```
(event_source="Microsoft-Windows-Security-Auditing" (event_id IN ["528", "529", "4624", "4625"] logon_type="2" ComputerName IN ["%ServerSystems%", "%DomainControllers%"])  -(logon_process="Advapi" ComputerName="%Workstations%"))
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?=.*(?:.*528|.*529|.*4624|.*4625))(?=.*2)(?=.*(?:.*%ServerSystems%|.*%DomainControllers%))))(?=.*(?!.*(?:.*(?=.*Advapi)(?=.*%Workstations%)))))'
```



