**Step(1) Install Nagios Plugins**\
Follow steps as given in link: https://support.nagios.com/kb/article/nagios-plugins-installing-nagios-plugins-from-source-569.html

**Step(2) Install NRPE**\
Follow steps as given in link: https://support.nagios.com/kb/article/nrpe-how-to-install-nrpe-v3-from-source-515.html

**Step(3)Verify NRPE connectivity from Nagios Monitoring Server**\
Login to Nagios Monitoring Server and issue command as follows:

`/usr/local/nagios/libexec/check_nrpe -H <linux_host_ip_address>`\
It should return output with NRPE Version.

**NOTE**: Make sure port 5666 is opened between Nagios Monitoring Server and Nagios hosts.


**Step(4)Add Linux Host to Nagios Monitoring Server**\
(i) To add a remote host you need to create a two new files “hosts.cfg” and “services.cfg” under “/usr/local/nagios/etc/” location.
Commands:-\
```
cd /usr/local/nagios
touch hosts.cfg
touch services.cfg
```

(ii) Now Open Config file `vi /usr/local/nagios/etc/nagios.cfg` and add below lines in the file.

```
cfg_file=/usr/local/nagios/etc/hosts.cfg
cfg_file=/usr/local/nagios/etc/services.cfg
```

(iii) Now open hosts.cfg fileusing command `vi /usr/local/nagios/etc/hosts.cfg` and add the default host template name and define remote hosts as shown below. Make sure to replace host_name, alias and address with your remote host server details.

```
## Default Linux Host Template ##
define host{
name                            linux-box               ; Name of this template
use                             generic-host            ; Inherit default values
check_period                    24x7        
check_interval                  5       
retry_interval                  1       
max_check_attempts              10      
check_command                   check-host-alive
notification_period             24x7    
notification_interval           30      
notification_options            d,r     
contact_groups                  admins  
register                        0                       ; DONT REGISTER THIS - ITS A TEMPLATE
}

## Default
define host{
use                             linux-box               ; Inherit default values from a template
host_name                       ansible-node1                   ; The name we're giving to this server
alias                           CentOS 7                ; A longer name for the server
address                         10.148.0.5            ; IP address of Remote Linux host
```

(iv) Next open services.cfg file (/usr/local/nagios/etc/services.cfg) and add the following services to be monitored.

```
define service{
        use                     generic-service
        host_name               ansible-node1
        service_description     CPU Load
        check_command           check_nrpe!check_load
        }

define service{
        use                     generic-service
        host_name               ansible-node1
        service_description     Total Processes
        check_command           check_nrpe!check_total_procs
        }

define service{
        use                     generic-service
        host_name               ansible-node1
        service_description     Current Users
        check_command           check_nrpe!check_users
        }

define service{
        use                     generic-service
        host_name               ansible-node1
        service_description     SSH Monitoring
        check_command           check_nrpe!check_ssh
        }

define service{
        use                     generic-service
        host_name               ansible-node1
        service_description     FTP Monitoring
        check_command           check_nrpe!check_ftp
```

(v) Now NRPE command definition needs to be created in commands.cfg file(/usr/local/nagios/etc/objects/commands.cfg).
Add below lines at the bottom of the file

```

###############################################################################
# NRPE CHECK COMMAND
#
# Command to use NRPE to check remote host systems
###############################################################################
define command{
        command_name check_nrpe
        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
        }
```

(vi) Finally, verify Nagios Configuration files for any errors, using command `/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg`

(vii) Restart Nagios using command `systemctl restart nagios`

That’s it. Now go to Nagios Monitoring Web interface at “http://Your-server-IP-address/nagios” or “http://FQDN/nagios” and Provide the username and password. Check that the Remote Linux Host was added and is being monitored.\

NOTE: For Object definitions details, please visit URL https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/3/en/objectdefinitions.html#service





