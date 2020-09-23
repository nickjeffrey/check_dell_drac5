# check_dell_drac5
nagios check for Dell DRAC5 service processor

Dell servers have a DRAC (Dell Remote Access Controller) card.  This is a lights-out management card, similar to the ILO (Integrated Lights Out) cards on HP intel-based servers, or the IMM (Integrated Management Module) on IBM intel-based servers, or the XClarity Controller on Lenovo servers.

In a nutshell, the DRAC is a tiny little dedicated processor running a BusyBox-based Linux implementation.  

The DRAC allows us to remotely power cycle a hung server, as well as telling us about the hardware status (ie failed fans, disks, etc)

This graphic shows where the DRAC is located on a Dell PowerEdge 1950.  You will notice that the DRAC has its own Ethernet interface.
<img src=https://github.com/nickjeffrey/check_dell_drac5/images/port.png>


You should create a low-privilege userid called “monitor” on the DRAC.  We will use this userid with the monitoring script.
Login to the web interface.
Click Remote Access, Configuration, Users.
Select an unused userid (3 in this example)

If you prefer using the command line to create the low-privilege userid, use these steps:

    ssh administrator@drac
    racadm config -g cfgUserAdmin -i 3 -o cfgUserAdminUserName monitor
    racadm config -g cfgUserAdmin -i 3 -o cfgUserAdminPassword monitor123
    racadm config -g cfgUserAdmin -i 3 -o cfgUserAdminPrivilege 0x00000001
    racadm config -g cfgUserAdmin -i 3 -o cfgUserAdminEnable 1
    racadm config -g cfgUserAdmin -i 3 -o cfgUserAdminIpmiLanPrivilege 2
    racadm config -g cfgUserAdmin -i 3 -o cfgUserAdminIpmiSerialPrivilege 15
    racadm config -g cfgUserAdmin -i 3 -o cfgUserAdminSolEnable 0
    exit

You will need a section similar to the following in the commands.cfg file on the nagios server.

      # 'check_dell_drac5' command definition
      define command {
             command_name    check_dell_drac5
             command_line    $USER1$/check_dell_drac5 $HOSTADDRESS$ $ARG1$ $ARG2$
             }



You will need a section similar to the following in the services.cfg file on the nagios server.

      # command format is "check_dell_drac5" if you want to use the username/password hardcoded in the script
      # to provide your own username/password, use "check_dell_drac5!username!password"
      define service {
              use                             generic-24x7-service
              host_name                       dellserver2-drac.example.com
              service_description             DRAC
              check_command                   check_dell_drac5
              }

