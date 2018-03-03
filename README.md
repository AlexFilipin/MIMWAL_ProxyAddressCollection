# MIMWAL ProxyAddressCollection Management

Microsoft Identity Manager documentation for ProxyAddressCollection management with a MIMWAL update resources workflow activity. The workflow is split into 4 steps and should be triggered by a MPR as soon as a ProxyAddressCollection relevant attribute is changed (e.g. MailNickName or MailDomain).

## Features
  1. Adds the old primary SMTP as secondary SMTP
  2. Removes the new primary SMPT if it's already in ProxyAddressCollection as secondary SMTP
  3. Checks if updates are truly required
  4. Manages mail addresses for two mail domains (both whith the same MailNickName)
  5. Error handling for differences between upper and lower case letters
  
## Worfklow Steps
  1. Build new values (mail addresses, sip address, UPN) - Save to WorkflowData
  2. Get current values from ProxyAddressCollection via iteration - Save to WorkflowData
  3. Determine required updates - Save to WorkflowData
  4. Update attributes on user

## Build new values
### Options
| Selected Option | Value |
| --- | --- |
| ActivityExecutionCondition | |
| ActorString | |
| ActorType | Service |
| Advanced | False |
| ApplyAuthorizationPolicy | False |
| Iteration | |
| QueryResources | False |
| ResolveDynamicGrammar | False |

### Updates
| Source Expression | Target | Allow Null |
| --- | --- | --- |
| `[//Target/MailNickname]` | `$MailNickname` | false |
| `[//Target/MailDomain]` | `$MailDomain` | false |
| `[//Target/SecondaryMailDomain]` | `$SecondaryMailDomain` | false |
| `IIF(IsPresent($MailDomain),$MailNickname + "@" + $MailDomain,Null())` | `$NewUPN` | false |
| `$NewUPN` | `[//WorkflowData/NewUPN]` | false |
| `IIF(IsPresent($NewUPN),"SMTP:" + $NewUPN,Null())` | `[//WorkflowData/NewPrimarySMTP]` | false |
| `IIF(IsPresent($NewUPN),"SIP" + ":" + $NewUPN,Null())` | `[//WorkflowData/NewPrimarySIP]` | false |
| `IIF(IsPresent($SecondaryMailDomain),"smtp:" + $MailNickname + "@" + $SecondaryMailDomain,Null())` | `[//WorkflowData/NewSecondarySMTP]` | false |
| `[//Target/MailNickname]` | `$MailNickname` | false |

## Get current values from ProxyAddressCollection
### Options
### Updates

## Determine required updates
### Options
### Updates

## Update attributes on user
### Options
### Updates
