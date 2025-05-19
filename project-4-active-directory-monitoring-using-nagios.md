# Project 4: Active Directory Health Checks using Nagios

## Introduction
In this project, I used Nagios , an open-source monitoring system, to keep an eye on the health and performance of Active Directory Domain Services (AD DS) . Nagios helped me detect and address issues in my IT infrastructure before they could impact business operations.

## Lab Setup and Tools I Used
- Windows Server with Active Directory Domain Services (AD DS)
- Nagios Core installed on a monitoring server
- NSClient++ installed on AD DS servers for monitoring

## Tool Installation

### Install Nagios Core
1. I Download Nagios Core from the official Nagios website.
2. I Follow the installation instructions for your operating system.
3. To start Nagios, I ran:
    ```bash
    sudo systemctl start nagios
    ```
4. I made sure Nagios would start automatically at boot:

    ```bash
    sudo systemctl enable nagios
    ```
5. Then, I accessed the Nagios web interface at http://<my_server_ip>/nagiosand logged in using the default credentials ( nagiosadmin).

### Install NSClient++
1. I downloaded NSClient++ from the official NSClient++ website.
2. I installed NSClient++ on all AD DS servers.
3. I configured the `nsclient.ini` file to allow NRPE (Nagios Remote Plugin Executor) and to collect necessary metrics:
    ```ini
    [/modules]
    NRPEListener = 1

    [/settings/NRPE/server]
    allow arguments = true
    insecure = true

    [/settings/NRPE/client]
    allowed hosts = <Nagios_Server_IP>

    [/settings/metrics]
    [/settings/log]
    ```

4. Finally, I restarted the NSClient++ service:
    ```powershell
    net stop nscp
    net start nscp
    ```

## Exercises

### Adding AD DS Servers to Nagios
**Steps:**
1. II edited the Nagios configuration file (`/usr/local/nagios/etc/nagios.cfg`) to include the new configuration directory:
    ```cfg
    cfg_dir=/usr/local/nagios/etc/servers
    ```
2. Create a new configuration file for the AD DS servers (`/usr/local/nagios/etc/servers/adds.cfg`) and add the following configuration:
    ```cfg
    define host {
        use                     windows-server
        host_name               adds-server
        alias                   AD DS Server
        address                 <AD_Server_IP>
    }

    define service {
        use                     generic-service
        host_name               adds-server
        service_description     CPU Load
        check_command           check_nrpe!check_cpu
    }

    define service {
        use                     generic-service
        host_name               adds-server
        service_description     Memory Usage
        check_command           check_nrpe!check_memory
    }

    define service {
        use                     generic-service
        host_name               adds-server
        service_description     AD Logon Events
        check_command           check_nrpe!check_eventlog!application!warning!error!4624
    }
    ```
3. Restart Nagios:
    ```bash
    sudo systemctl restart nagios
    ```

**Expected Output:**
- Nagios is configured to monitor AD DS servers and their services.

###  Monitoring AD Logon Events
**Steps:**
1. Ensure NSClient++ is configured to collect Windows Event Logs.
2. Add the following command definition to `commands.cfg` in the Nagios configuration directory:
    ```cfg
    define command {
        command_name    check_eventlog
        command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_eventlog -a 'application' 'warning' 'error' '$ARG1$'
    }
    ```
3. Update the service definition in `adds.cfg` to monitor logon events:
    ```cfg
    define service {
        use                     generic-service
        host_name               adds-server
        service_description     AD Logon Events
        check_command           check_eventlog!4624
    }
    ```

**Expected Output:**
- A service in Nagios displays logon events for the AD DS server.

### Setting Up Alerts for AD Performance Issues
**Steps:**
1. Edit the Nagios contacts configuration file (`/usr/local/nagios/etc/objects/contacts.cfg`) to configure email notifications.
2. Define alert conditions in `adds.cfg`:
    ```cfg
    define service {
        use                     generic-service
        host_name               adds-server
        service_description     CPU Load
        check_command           check_nrpe!check_cpu
        notifications_enabled   1
        contacts                nagiosadmin
    }
    ```
3. Restart Nagios to apply the changes:
    ```bash
    sudo systemctl restart nagios
    ```

**Expected Output:**
- Alerts are set up to notify administrators of potential AD performance issues via email.

###  Creating a Dashboard for AD Metrics
**Steps:**
1. Log in to the Nagios web interface.
2. Navigate to "Tactical Overview" to view an overview of monitored services.
3. Click on "Service Details" to see detailed metrics for each service.
4. Arrange and customize the view to create a comprehensive dashboard for AD metrics.

**Expected Output:**
- A dashboard displaying real-time AD metrics is available in the Nagios web interface.

###  Visualizing AD Security Metrics
**Steps:**
1. I configured NSClient++ to collect extra security-related metrics.
2. Add service definitions in `adds.cfg` to monitor security events (e.g., failed logon attempts, account lockouts):
    ```cfg
    define service {
        use                     generic-service
        host_name               adds-server
        service_description     Failed Logon Attempts
        check_command           check_eventlog!4625
    }

    define service {
        use                     generic-service
        host_name               adds-server
        service_description     Account Lockouts
        check_command           check_eventlog!4740
    }
    ```
3. Restart Nagios to apply the changes:
    ```bash
    sudo systemctl restart nagios
    ```

**Expected Output:**
- Services in Nagios display key AD security metrics, providing a comprehensive security overview.

---
## Conclusion
By completing this project, I successfully built an Active Directory monitoring system using Nagios and NSClient++ . I gained valuable experience in setting up health checks, tracking logon and security events, and configuring real-time alerts to keep my AD DS environment secure and performing well.
