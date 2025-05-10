
# Windows OpenSSH Client:
 This is the built-in SSH client in Windows that allows you to connect to remote SSH servers. When you use commands like ssh user@remote_host, you are using the SSH client. It utilizes your private keys to authenticate with the server.

#Windows OpenSSH Server:
 This is an optional feature in Windows that allows your Windows machine to act as an SSH server, so others can connect to it securely using SSH.

SSH Agent (OpenSSH Authentication Agent):#
 This is the component that securely holds your private SSH keys in memory after you've added them. Once the keys are loaded into the agent, you don't need to provide the passphrase each time you connect to a server that uses those keys for authentication. The agent handles the authentication behind the scenes. On Windows, the OpenSSH Authentication Agent service provides this functionality. You typically interact with it using commands like ssh-add to add keys.











https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement



Host key generation
Public keys have specific access control list (ACL) requirements that, on Windows, equate to only allowing access to administrators and the System user. The first time the sshd service is used, the key pair for the host is automatically generated.

 Important

You need to install OpenSSH Server before you can run the commands in this article. For more information, see Get started with OpenSSH for Windows.

By default, you need to start sshd manually. To configure it to start automatically each time the server is restarted, run the following commands from an elevated PowerShell prompt on your server:

PowerShell

Copy
# Set the sshd service to be started automatically.
Get-Service -Name sshd | Set-Service -StartupType Automatic

# Start the sshd service.
Start-Service sshd
Because there's no user associated with the sshd service, the host keys are stored under C:\ProgramData\ssh.



PowerShell

Copy
# By default, the ssh-agent service is disabled. Configure it to start automatically.
# Run the following command as an administrator.
Get-Service ssh-agent | Set-Service -StartupType Automatic

# Start the service.
Start-Service ssh-agent

# The following command should return a status of Running.
Get-Service ssh-agent

# Load your key files into ssh-agent.
ssh-add $env:USERPROFILE\.ssh\id_ecdsa
After you add the key to the ssh-agent service on your client, the ssh-agent service automatically retrieves the local private key and passes it to your SSH client.




# configs located at 
"C:\Windows\System32\OpenSSH"




```powershell
Get-Content -Path "$env:USERPROFILE\.ssh\config"
```
