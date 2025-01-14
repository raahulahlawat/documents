## **Monitoring and Alerting for Critical Commands Execution**

### **Objective:**

The goal is to monitor the execution of critical commands (**`rm`, `shutdown`, `sudo`, `apt-get`**) on a Linux server and send email alerts with detailed information about the execution of those commands.

### **Setup Steps:**

#### **1\. Install `msmtp` for Email Notifications:**

To send email alerts when critical commands are executed, we use `msmtp`, a simple mail transfer agent.

##### **Install `msmtp`:**

Run the following command to install `msmtp` and `msmtp-mta`:

| sudo apt install msmtp msmtp-mta |
| :---- |

#### **2\. Configure Email Settings in `msmtp`:**

Create and configure the `.msmtprc` file to use your email provider's SMTP settings.

**File location**: `~/.msmtprc`

* **File content** (example configuration for Gmail):

| account defaulthost smtp.gmail.comport 587from raahulahlawat@gmail.comauth onuser raahulahlawat@gmail.compassword swjpwzsxcycdmjantls ontls\_certcheck offtls\_starttls onlogfile \~/.msmtp.log |
| :---- |

**Important:** Make sure to set the correct file permissions to protect your email credentials:

| chmod 600 \~/.msmtprc |
| :---- |

#### **3\. Set Up Audit Rules:**

To monitor the execution of critical commands like `rm`, `shutdown`, `sudo`, and `apt-get`, we will set up audit rules.

##### **Add audit rules:**

Create or edit the file `/etc/audit/rules.d/audit.rules` with the following contents:

| \-w /bin/rm \-p x \-k critical\_command\-w /sbin/shutdown \-p x \-k critical\_command\-w /usr/bin/sudo \-p x \-k critical\_command\-w /usr/bin/apt-get \-p x \-k critical\_command |
| :---- |

This will monitor the execution of these commands and trigger alerts whenever they are executed.

##### **Apply the audit rules:**

After updating the rules, apply them by running the following command:

| sudo auditctl \-R /etc/audit/rules.d/audit.rules |
| :---- |

#### **4\. Create a Monitoring Script to Send Alerts:**

Create a script that will continuously monitor the audit log for critical commands and send an email alert when such commands are executed.

##### **Script Location: `/usr/local/bin/critical_alert.sh`**

##### 

##### 

##### 

##### 

##### 

##### **Script Content:**

| \#\!/bin/bashLOG\_FILE="/var/log/audit/audit.log"ALERT\_EMAIL="rahul.ahlawat@fosteringlinux.com"tail \-Fn0 "$LOG\_FILE" | while read line; do    echo "$line" | grep \-i "critical\_command" \> /dev/null    if \[ $? \= 0 \]; then        USER=$(echo "$line" | grep \-oP 'uid=\\K\[0-9\]+')        COMMAND=$(echo "$line" | grep \-oP 'exe=\\"\\K\[^\\"\]+')        FILE\_PATH=$(echo "$line" | grep \-oP 'name=\\K\[^\\s\]+')        echo "USER: $USER"        echo "COMMAND: $COMMAND"        echo "FILE\_PATH: $FILE\_PATH"        USERNAME=$(getent passwd "$USER" | cut \-d: \-f1)        if \[ \-z "$COMMAND" \]; then            COMMAND="Unknown Command"        fi        \# Fetch the IP address and username of the user who is logged in        IP=$(who | grep "$USERNAME" | awk '{print $5}' | tr \-d '()' | head \-n 1\)        USERNAME\_FROM\_IP=$(who | grep "$IP" | awk '{print $1}' | head \-n 1\)        SERVER\_IP=$(hostname \-I | awk '{print $1}')        TIME\_EXECUTION=$(date)        EMAIL\_BODY="User Name : $USERNAME\_FROM\_IPLogin From IP : $IPLogin to Server IP : $SERVER\_IPCommand executed : $COMMAND $FILE\_PATHTime of execution : $TIME\_EXECUTION"        echo "$EMAIL\_BODY" | msmtp "$ALERT\_EMAIL"    fidone |
| :---- |

##### 

##### **Make the Script Executable:**

Ensure that the script has execute permissions:

| sudo chmod \+x /usr/local/bin/critical\_alert.sh |
| :---- |

#### **5\. Run the Monitoring Script:**

To start monitoring the audit log, run the script:

| nohup /usr/local/bin/critical\_alert.sh & |
| :---- |

The script will keep running and will send an email alert whenever a critical command is executed.

### **Verification Steps:**

#### **Verify Audit Rules:**

You can check if the audit rules are applied successfully by running the following command:

| sudo auditctl \-l |
| :---- |

This will list the active audit rules.

#### **Test Email Functionality:**

To verify that `msmtp` is correctly configured and can send emails, try sending a test email:

| echo "Test email" | msmtp your\_email@example.com |
| :---- |

If the test email is sent successfully, the email configuration is working.

#### **Test Command Monitoring:**

Run one of the critical commands (`sudo`, `rm`, `shutdown`, or `apt-get`) to see if the alert is triggered.

For example, run the following command:

| sudo rm \-rf /tmp/testfile |
| :---- |

Check your email inbox to ensure that the alert has been received.