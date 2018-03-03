MIMWAL ProxyAddressCollection Management
=============

Microsoft Identity Manager documentation for ProxyAddressCollection management with a MIMWAL update resources workflow activity

Worfklow Steps
-------
The workflow is split into 4 steps and should be triggered by a MPR as soon as a ProxyAddressCollection relevant attribute is changed (e.g. MailNickName or MailDomain). In this example we are also updating the users UPN:

  1. Build new values (mail addresses, UPN) - Save to WorkflowData
  2. Get current values from ProxyAddressCollection via iteration - Save to WorkflowData
  3. Determine which values need to be updated - Save to WorkflowData
  4. Update attributes on user
  
Features
-------
  1. We will add the old primary SMTP as secondary SMTP
  2. We will remove the new primary SMPT if it's already in ProxyAddressCollection as secondary SMTP
  3. We will check if updates are truly required
