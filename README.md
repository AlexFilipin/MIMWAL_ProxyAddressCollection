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
| Selected Option | Value |
| --- | --- |
| ActivityExecutionCondition | |
| ActorString | |
| ActorType | Service |
| Advanced | True |
| ApplyAuthorizationPolicy | False |
| Iteration | [//Target/ProxyAddressCollection] |
| QueryResources | False |
| ResolveDynamicGrammar | False |
### Updates
| Source Expression | Target | Allow Null |
| --- | --- | --- |
| `IIF(Eq(Left([//Value],5),"SMTP:",true),[//Value],Null())` | `[//WorkflowData/OldPrimarySMTP]` | false |
| `IIF(Eq(Left([//Value],4),"SIP:",true),[//Value],Null())` | `[//WorkflowData/OldPrimarySIP]` | false |
| `IIF(Eq([//WorkflowData/NewPrimarySMTP],[//Value]),[//Value],Null())` | `[//WorkflowData/PrimarySmtpToRemove]` | false |
| `IIF(Eq([//WorkflowData/NewPrimarySIP],[//Value]),[//Value],Null())` | `[//WorkflowData/PrimarySipToRemove]` | false |
### Comment
1. Iteration over ProxyAddressCollection
2. The search for old primary values is case sensitive
2. The search if the new values are already in the ProxyAddressCollection is case insensitive

## Determine required updates
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
| `Not(Eq("SMTP:"+[//WorkflowData/NewUPN],[//WorkflowData/OldPrimarySMTP]))` | `[//WorkflowData/UpdatePrimarySMTP]` | false |
| `Not(Eq("SIP:"+[//WorkflowData/NewUPN],[//WorkflowData/OldPrimarySIP]))` | `[//WorkflowData/UpdatePrimarySIP]` | false |
| `Not(Contains([//Target/ProxyAddressCollection],[//WorkflowData/NewSecondarySMTP]))` | `[//WorkflowData/UpdateSecondarySMTP]` | false |
### Comment
You will notice I build the new primary SMTP and SIP again instead of using my already populated value in WorkflowData. I have seen issues during testing where the case sensitive was not maintained.
## Update attributes on user
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
| `[//WorkflowData/NewUPN]` | `[//Target/Upn]` | false |
| `[//WorkflowData/NewUPN]` | `[//Target/Email]` | false |
| `IIF([//WorkflowData/UpdatePrimarySMTP],RemoveValues([//WorkflowData/OldPrimarySMTP]),Null())` | `[//Target/ProxyAddressCollection]` | false |
| `IIF([//WorkflowData/UpdatePrimarySMTP],RemoveValues([//WorkflowData/PrimarySmtpToRemove]),Null())` | `[//Target/ProxyAddressCollection]` | false |
| `IIF([//WorkflowData/UpdatePrimarySMTP],InsertValues("SMTP:"+[//WorkflowData/NewUPN]),Null())` | `[//Target/ProxyAddressCollection]` | false |
| `IIF([//WorkflowData/UpdatePrimarySMTP],InsertValues(ReplaceString([//WorkflowData/OldPrimarySMTP],"SMTP:","smtp:")),Null())` | `[//Target/ProxyAddressCollection]` | false |
| `IIF([//WorkflowData/UpdatePrimarySIP],RemoveValues([//WorkflowData/OldPrimarySIP]),Null())` | `[//Target/ProxyAddressCollection]` | false |
| `IIF([//WorkflowData/UpdatePrimarySIP],RemoveValues([//WorkflowData/PrimarySipToRemove]),Null())` | `[//Target/ProxyAddressCollection]` | false |
| `IIF([//WorkflowData/UpdatePrimarySIP],InsertValues("SIP:"+[//WorkflowData/NewUPN]),Null())` | `[//Target/ProxyAddressCollection]` | false |
| `IIF([//WorkflowData/UpdateSecondarySMTP],InsertValues([//WorkflowData/NewSecondarySMTP]),Null())` | `[//Target/ProxyAddressCollection]` | false |
