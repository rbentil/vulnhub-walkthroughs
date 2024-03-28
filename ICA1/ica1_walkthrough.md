# ICA1 Walkthrough

## Initial Enumeration

1. **NMAP Scan**: Conducted an extensive NMAP scan to identify open ports and services running on the target machine. This helps in understanding the attack surface and potential entry points.
   ```bash
   nmap -sV -sC -T4 -p- -A <target_ip>
   ```

   Open Ports:
   - Port 22 (SSH)
   - Port 80 (HTTP)
   - Port 3306 (MySQL)
   - Port 33060 (MySQL)

2. **Exploring Web Application**: Upon navigating to the HTTP site, discovered a qdPM login page, indicating the presence of a project management system powered by qdPM. Noticed qdPM was running version 9.2.

## Exploitation

3. **Vulnerability Assessment**: Used `searchsploit` to find vulnerabilities associated with qdPM version 9.2. This step is crucial for identifying potential exploits that can be leveraged to gain unauthorized access.
   ```bash
   searchsploit qdPM 9.2
   ```

   Identified vulnerabilities:
   - Cross-site Request Forgery (CSRF)
   - Password Exposure (Unauthenticated)

4. **Exploiting Database Configuration Exposure**: Exploited the password and database connection string exposure vulnerability in qdPM 9.2 by accessing the `databases.yml` file. This file often contains sensitive information like database credentials.
   - Located at `http://<website>/core/config/databases.yml`

   Extracted credentials:
   - Username: qdpmadmin
   - Password: UcVQCMQk2STVeS6J

5. **Accessing MySQL Database**: Leveraged the harvested credentials to access the MySQL database. This step allows for database enumeration to gather additional information or escalate privileges.
   ```bash
   mysql -u qdpmadmin -p -h <target_ip>
   ```

   Conducted database enumeration:
   - Identified users and login details.

6. **Decoding Encrypted Passwords**: Decoded base64-encoded passwords from the database to gain access to additional accounts. This is crucial for expanding the attack surface and obtaining more credentials.

7. **Brute-Forcing SSH Credentials**: Utilized Hydra to perform a brute-force attack against SSH using harvested usernames and passwords. This helps in gaining access to the target system via SSH.
   ```bash
   hydra -L user.dict -P pass.dict ssh://<target_ip>
   ```

## Privilege Escalation

8. **Accessing User Account**: Successfully logged in via SSH using discovered credentials. This step provides access to the user's environment and files.
   ```bash
   ssh travis@<target_ip>
   ```

9. **User Enumeration**: Explored user's home directory to retrieve user flag. This helps in escalating privileges and obtaining additional information.
   ```bash
   cat ~/user.txt
   ```

10. **Escalating Privileges**: Utilized the `strings` utility to inspect an executable file with SUID permissions (`get_access`) and discovered a hidden file path leading to system information. This step is crucial for privilege escalation.
    ```bash
    strings /opt/get_access
    ```

11. **Creating a Backdoor**: Created a custom `cat` binary in the `/tmp` directory that executes a shell. This backdoor allows for easy access to the system.
    ```bash
    echo '/bin/bash' > /tmp/cat; chmod +x /tmp/cat
    ```

12. **Modifying PATH Variable**: Edited the system's PATH variable to prioritize the `/tmp` directory. This ensures that the custom `cat` binary is executed instead of the original `cat`.
    ```bash
    export PATH=/tmp:$PATH
    ```

13. **Executing Backdoor**: Executed the `get_access` binary to escalate privileges to root. This step grants root access to the attacker.
    ```bash
    /opt/get_access
    ```

## Final Stage

14. **Root Access**: Accessed the root account and obtained the root flag. This signifies successful completion of the attack.
    ```bash
    cat /root/system.info
    ```

## Conclusion

By following this step-by-step walkthrough, the ICA1 box on Vulnhub was successfully completed. Each step was carefully executed to identify vulnerabilities, exploit them, and escalate privileges to gain unauthorized access to the target system. This exercise highlights the importance of thorough enumeration, vulnerability assessment, and exploitation techniques in penetration testing scenarios.
```
