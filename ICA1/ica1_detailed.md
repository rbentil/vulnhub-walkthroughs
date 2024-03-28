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
   
   These open ports imply the presence of a web server running (Port 80), which is most likely connected to the databases on port 3306 and 33060, which imply MySQL services. Port 22 would most likely be very useful to us as we progress. Most CTFs often begin with exploiting a vulnerability in a web application to gain access into the system. The nmap scan also tells us this system is running on Linux.

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
   
   We retrieved this information from the databases.yml file. It exposes a user 'qdpmadmin' and a password <?php echo urlencode('UcVQCMQk2STVeS6J') ; ?> .
   The PHP code <?php echo urlencode('UcVQCMQk2STVeS6J') ; ?> is a snippet that generates a URL-encoded version of the string 'UcVQCMQk2STVeS6J'.

Here's what each part of the code does:

So, the code <?php echo urlencode('UcVQCMQk2STVeS6J') ; ?> will output the URL-encoded version of the string 'UcVQCMQk2STVeS6J'.

**EXTRA NOTES**

What is the difference between encryption and encoding?

   Encoding:
   - Purpose: Encoding is primarily used to represent data in a specific format that is suitable for transmission or storage. It is not intended to hide the data or provide security.
   - Transformation: Encoding converts data from one format to another format using predefined rules or algorithms. It does not require a key for transformation and is typically reversible.
   - Examples: Common encoding schemes include Base64, URL encoding, ASCII, Unicode, and HTML encoding. These schemes are used to represent binary data as text or to ensure compatibility with different systems.
   - Use Cases: Encoding is commonly used in data transmission over networks, in data storage, and in web development to handle special characters or non-textual data.

   Encryption:
   - Purpose: Encryption is used to secure sensitive data by converting it into an unintelligible form, making it unreadable without the appropriate decryption key.
   - Transformation: Encryption transforms data using an encryption algorithm and a secret key. The encrypted data, or ciphertext, is designed to be decrypted only by authorized parties who possess the key.
   - Security: Encryption provides confidentiality and protects data from unauthorized access or interception. It ensures that even if the encrypted data is intercepted, it remains secure without the decryption key.
   - Examples: Common encryption algorithms include AES (Advanced Encryption Standard), RSA, and DES (Data Encryption Standard). These algorithms use mathematical techniques to scramble the data into ciphertext.
   - Use Cases: Encryption is used to secure communication channels (e.g., HTTPS for web browsing), protect sensitive information (e.g., passwords, financial transactions), and safeguard data at rest (e.g., encrypted files or databases).

   Extracted credentials:
   - Username: qdpmadmin
   - Password: UcVQCMQk2STVeS6J

5. **Accessing MySQL Database**: Leveraged the harvested credentials to access the MySQL database. This step allows for database enumeration to gather additional information or escalate privileges. 
   
   While it is true that the login page from the webapp requests for an email and password, having access to database credentials will be extremely useful since we enumerated open MySQL ports. We can use the MySQL utility in Linux to login as qdpmadmin with the credentials we harvested and potentially access the company's database.

   Linux has a command line utility we can use to access database servers. It's called mysql. It's not the only database command line utility, however. Other examples of database command line utilities are: 
    - PostgreSQL: The psql command-line client is commonly used to interact with PostgreSQL database servers.
    - SQLite: The sqlite3 command-line client is used to interact with SQLite databases.
    - MongoDB: The mongo command-line client is used to interact with MongoDB databases.
    - Redis: The redis-cli command-line client is used to interact with Redis databases.
    
   So, different databases have different programs we can use to access their respective database servers.

   These are the most common options to get started with using the mysql utility:
   - Specifying Host and Port:
        - -h, --host=<hostname>: Specify the hostname of the MySQL server.
        - -P, --port=<port>: Specify the port number of the MySQL server.

   - Authentication:
        - -u, --user=<username>: Specify the MySQL username to connect with.
        - -p, --password[=<password>]: Prompt for the MySQL password. If <password> is not provided, the user will be prompted to enter it.

   - Database Operations:
        - -D, --database=<dbname>: Connect to the specified database when connecting to the MySQL server.
        - -e, --execute=<statement>: Execute the specified SQL statement.
        
   In this case, we're using mysql; (from the open 3306 and 33060 ports from our nmap enumeration.)

   ```bash
   mysql -u qdpmadmin -p -h <target_ip>		
   ```

   In this command, we want to log into the MySQL database located on the 192.168.56.107 server using the username of qdpmadmin. Since we specified the -p option and did not provide a password, we will be prompted to enter one.  

   Conducted database enumeration:
   - Identified users and login details.
   
   Enter the command:

   ```bash
   show databases;
   ```

   after you successfully log into the database server to show the available databases on the server. We could have directly entered the specific database we wanted to connect to from the command line utility during our login process, but that would not have been feasible since we need to do manual enumeration of the available databases. We also didn't know the available databases on the server.

   The databases available on the server are:
   - information_schema
   - mysql
   - performance_schema
   - qdpm
   - staff
   - sys

   Manual enumeration of each database will be beneficial, but the "staff" database looks glaringly useful. 

   Next, use the command:

   ```bash
   use staff;
   ```

   to use the "staff" database. Make sure to end all of your commands with a semicolon.

   ```bash
   show tables;
   ```

   This command will show all tables in the "staff" database. This presents us with the following tables:
   - department
   - login
   - user
   
   We can harvest vital credentials from the "user" table by using:

   ```SQL
   select * from user;
   ```

   to view the available users. Each of these users iares likely to be users on the system itself so make sure to add them to your notes.

   We can harvest passwords from the login table using the command below

   ```sql
   select * from login;
   ``` 

   to view likely login credentials. We can straight away notice that these passwords are encrypted in base64. How can I tell?
    
   > Base64 is a binary-to-text encoding scheme that represents binary data in an ASCII string format.
   Base64 encoding is commonly used to encode binary data, such as images, audio files, or cryptographic keys, into a format that can be safely transmitted over text-based protocols or stored in text-based formats without corruption.
   
   1. Length and Character Set: Base64 encoded strings typically consist of a series of alphanumeric characters, often including the characters A-Z, a-z, 0-9, as well as two additional characters (usually + and /) for encoding. The length of the string is usually a multiple of 4 characters.
   
   2. Padding: Base64 encoding pads the end of the string with one or two = characters to indicate the number of missing characters needed to make the length of the encoded string a multiple of 4. In this case, the presence of the double == at the end suggests that padding is used.
   
   Note: Encoding is not the same as encryption.
    
   Purpose:
   - Encoding: Encoding is primarily used to represent data in a specific format that is suitable for transmission or storage. It is not intended to hide the data or provide security. Encoding schemes are reversible and do not require a key for transformation.
   - Encryption: Encryption is used to secure sensitive data by converting it into an unintelligible form, making it unreadable without the appropriate decryption key. Encryption provides confidentiality and protects data from unauthorized access or interception.
    
   Transformation:
   - Encoding: Encoding transforms data from one format to another format using predefined rules or algorithms. It typically involves converting data into a different representation, such as converting binary data into text-based formats or vice versa. Encoding does not involve any form of secrecy or security.
   - Encryption: Encryption transforms data using an encryption algorithm and a secret key. The encrypted data, or ciphertext, is designed to be decrypted only by authorized parties who possess the key. Encryption involves scrambling the data in such a way that it becomes unreadable without the decryption key.
    
   Security:
   - Encoding: Encoding does not provide any security features. It is reversible, and the original data can be recovered by decoding the encoded representation using the appropriate decoding algorithm or scheme.
   - Encryption: Encryption provides security by ensuring that encrypted data cannot be understood or deciphered without the decryption key. It protects data confidentiality and prevents unauthorized access to sensitive information.
   
   Reversibility:
   - Encoding: Encoding schemes are reversible, meaning that the original data can be recovered by decoding the encoded representation using the appropriate decoding algorithm or scheme.
   - Encryption: Encryption is intended to be irreversible without the appropriate decryption key. While the original data can be recovered by decrypting the ciphertext with the decryption key, without the key, the ciphertext remains unreadable.

   In summary, encoding is used for data representation and formatting, while encryption is used for data security and confidentiality. Encoding is reversible and does not provide security features, whereas encryption is irreversible without the appropriate decryption key and provides security through data confidentiality.

6. **Decoding Encoded Passwords**: Decoded base64-encoded passwords from the database to gain access to additional accounts. This is crucial for expanding the attack surface and obtaining more credentials. We can use the `base64` command-line tool to decode the encoded texts.

   ```bash 
   echo 'encodedText' | base64 -d
   ```

   Replace `encodedText` with the encoded texts.

7. **Brute-Forcing SSH Credentials**: Utilized Hydra to perform a brute-force attack against SSH using harvested usernames and passwords. This helps in gaining access to the target system via SSH.

   What is Hydra?
   Hydra is a popular tool used for performing brute-force attacks on various login systems. It works by systematically attempting to log in to a target system using a combination of usernames and passwords from a specified list (often referred to as a dictionary). Here's how Hydra works:

   - Configuration: First, the user provides configuration parameters to specify the target system and the login protocol to be used. This includes specifying the target host, port, protocol (e.g., HTTP, SSH, FTP), usernames, passwords, and any additional options relevant to the login process.

   - Dictionary Attack: Hydra operates on the principle of a dictionary attack, where it systematically tries each username-password combination from a provided list (dictionary) until a successful login is found or the entire dictionary is exhausted. The dictionary typically contains common usernames and passwords, as well as variations and combinations of these.

   - Response Analysis: Hydra analyzes the responses received from the target system after each login attempt. Depending on the response, it can determine whether the login attempt was successful, failed due to incorrect credentials, or was blocked by security measures such as account lockouts or CAPTCHA challenges.

   - How does Hydra attempt SSH logins without the private SSH key?
   
   Hydra attempts SSH logins without needing the private SSH key because it operates as a brute-force tool, not as an SSH client. Here's how it works:

   - SSH Protocol: SSH (Secure Shell) is a protocol used for secure remote access to systems. When logging in via SSH, the client (such as Hydra) sends the username and password to the SSH server for authentication.

   - Brute-Force Attack: Hydra performs a brute-force attack by systematically trying different combinations of usernames and passwords against the SSH server. It does this by sending login attempts directly to the SSH server's authentication mechanism.

   - No Private Key Required: Unlike traditional SSH clients, Hydra does not establish an SSH session or perform key-based authentication. Instead, it directly sends authentication requests containing usernames and passwords to the SSH server, mimicking the behavior of a legitimate client attempting to log in.

   - Response Analysis: After each login attempt, Hydra analyzes the response received from the SSH server. If the response indicates a successful login (such as a successful authentication message), Hydra marks the attempt as successful. Otherwise, it continues trying different combinations until either a successful login is found or the entire dictionary of credentials is exhausted.

   - When you attempt to log in via SSH with incorrect credentials, the SSH server will typically still prompt you for a password, even if the credentials are incorrect. This behavior is part of the SSH protocol's security mechanisms, which aim to prevent an attacker from distinguishing between valid and invalid usernames during the authentication process.


   ```bash
   hydra -L user.dict -P pass.dict ssh://<target_ip>
   ```

   The `-L` option is used to provide a file of users and `-P` provides a dictionary file with a list of passwords.

   Hydra returned two valid credentials for us to SSH into the system:
   ```
   [22][ssh] host: 192.168.56.107   login: dexter   password: 7ZwV4qtg42cmUXGX
   [22][ssh] host: 192.168.56.107   login: travis   password: DJceVy98W28Y7wLg
   1 of 1 target successfully completed, 2 valid passwords found
   ```

## Privilege Escalation

8. **Accessing User Account**: Successfully logged in via SSH using discovered credentials. This step provides access to the user's environment and files.

   ```bash
   ssh travis@<target_ip>
   ```

9. **User Enumeration**: Explored the user's home directory to retrieve the user flag. This helps in escalating privileges and obtaining additional information.

   ```bash
   cat ~/user.txt
   ```

   We captured the user flag in Travis' home directory. We can try to `cd` into Dexter's home directory, but we have a permission denied output.

   Log in as Dexter to have access to his home directory and list the files there. We find a `user.txt` file with information. The note reads:

   ```
   It seems to me that there is a weakness while accessing the system.
   As far as I know, the contents of executable files are partially viewable.
   I need to find out if there is a vulnerability or not.
   ```

   With this information, we would have to do manual enumeration to find the executable. We find a `get_access` file in the `/opt` directory:

   ```
   -rwsr-xr-x  1 root root 16816 Sep 25  2021 get_access
   ```

   Viewing the permissions of this file tells us we can execute the file. Since `get_access` is a binary file, we can't read its contents. The note said some of the contents of executable files are partially readable, giving us a clue as to how to read the contents of this file. We can use the `strings` tool. The `strings` tool allows us to view the partially readable parts of a system binary.

10. **Escalating Privileges**: Utilized the `strings` utility to inspect the executable file (`get_access`) and discovered a hidden file path leading to system information. This step is crucial for privilege escalation.

    ```bash
    strings /opt/get_access
    ```

A line from the `get_access` access file reads `cat root/systeminfo`. We can leverage this little detail to create a backdoor by creating a custom `cat` binary. When we execute the `get_access` file, this command will also be executed along with it, allowing us to get root access.

11. **Creating a Backdoor**: Created a custom `cat` binary in the `/tmp` directory that executes a shell. This backdoor allows for easy access to the system. We do this because one of the lines in the `get_access` file reads `cat /root/system.info`. We can leverage this for our privilege escalation by creating a custom `cat` binary and modifying the `PATH` variable.

    ```bash
    echo '/bin/bash' > /tmp/cat; chmod +x /tmp/cat
    ```

    This code copies the contents of the bash terminal into the custom `cat` binary we have created. This means that when we enter the `cat` command, it will actually launch a bash terminal instead of the original `cat` command.

12. **Modifying PATH Variable**: Edited the system's `PATH` variable to prioritize the `/tmp` directory. This ensures that the custom `cat` binary is executed instead of the original `cat`.

    The `PATH` variable tells the kernel where in the system to look when executing a command/binary. By default, the `PATH` variable is set to:

    ```
    /usr/local/bin:/usr/bin:/bin:/usr/local/games
    ```

    We can view this by typing `$PATH` into our command line.

    ```bash
    export PATH=/tmp:$PATH
    ```

    This command tells the system to first look into the `/tmp` directory, then the original `PATH` for system binaries.

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
