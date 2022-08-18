- 👋 Hi, I’m @murthy419
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
murthy419/murthy419 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

 PowerHA  migration -  from PowerHA  7.1.3.5     to    PowerHA  (7.2.2.1)                                          

 

 

              High level steps  -  Migrate PowerHA on both nodes  (NOT rolling upgrade)                                                       

                                                                       

1            Take snaps and mksysb and copy mksysb on NIM                                                         

                                                                       

2            Repository disk -   5 GB SHARED                                                           

                                                                       

3            Powerha software copy on both nodes from nim server                                             

                                                                       

4            Create Cluster snapshot                                           

                                                                       

5            Clone rootvg  on both nodes                                                  

                                                                       

6            App/ DBA team - stop services                                             

                                                                       

7            Stop PowerHA on Both nodes                                                

                                                                       

8            Reboot  both servers with cloned  disk                                               

                                                                       

9            Check below on both nodes and make sure its 7.2 TL2 SP2 and Powerha 7.1.3 SP5                                                     

                halevel -s                                                     

                oslevel -s                                                     

                                                                       

10          Commit any software which are in applied state                                            

                installp -c all                                                

                                                                        

11          PowerHA software  untar                                                       

                   tar -xvf /sysbkup/PowerHA_72_mig.tar                                                      

                                                                       

                                                                       

12          check "rhosts" file   (on all nodes)                                                       

              Add IP's of all hosts  in  /etc/cluster/rhosts                                                    

                                                                       

14          netsvc.conf file  (on all nodes)                                               

                  cp -p /etc/netsvc.conf /etc/netsvc.conf.prehamig                                                    

                                                                       

              Make sure on below entry is there in netsvc.conf and all remaining lines should be removed.                                                      

                        hosts = local, bind4                                                        

                                                                       

                                                                       

                                                                       

                x) uncomment below one from /etc/inetd.conf                                           

                            caa_cfg stream  tcp6    nowait  root    /usr/sbin/clusterconf clusterconf >>/var/adm/ras/clusterconf.log 2>&1                                                       

                                                                       

                                                                       

16          Reboot all cluster nodes                                                          

                                                                       

17          Make sure below servers are running on all nodes in cluster and let it come by itself after reboot. or re-check step 14.                                                       

                                                                       

              $ lssrc -a | grep -i caa                                                

              clcomd           caa              6488274      active                                                     

              clconfd          caa              13697102     active                                                     

                                                                       

18          Install efix  - IJ04268s2a.180403.AIX72TL02SP02.epkg.Z  No reboot needed for this efix                                    

                                                                       

19          PowerHA upgrade  to 7.2.2.1  and reboot node on first node                                                 

                                                                       

                                                                       

20          PowerHA upgrade  to 7.2.2.1  and remaining nodes,  one node at a time                                                 

                                                                       

                                                                       

                                                                       

21          Set "Anti-Collocation" for Network  to "Disable Firstalias"                                                         

                                                                       

              smitty hacmp-->Cluster Applications and Resources-->Resources-->Configure Service IP Labels/Address Distribution Preference-->Select the Network(net_ether_01)                                           

                    >  Cluster Applications and Resources                                                        

                       > Resources                                                        

                            > Configure Service IP Labels/Addresses                                            

                                 > Configure Service IP Labels/Address Distribution Preference                                                    

                                               >  select  (net_ether_01)                                                    

                                                             Network Name                                          :  net_ether_01                                 

                                                                   Distribution preference                                              :  Disable Firstalias                                      (Note:  if before this value was "Anti-Collocation" , change to "Disable Firstalias

                                                                       

22          Check Policy                                                   

                                                                        

               clRGinfo -v        check these are correct                              

                      Startup Policy   : Online On Home Node Only                                                        

                     Fallover Policy  : Fallover To Next Priority Node In The List                                                 

                      Fallback Policy  : Never Fallback                                                   

                                                                       

23          Sync cluster from node1 to all nodes                                                  

                                                                       

              smitty hacmp                                              

                Custom Cluster Configuration                                             

                       Verify and Synchronize Cluster Configuration (Advanced)                                                

                                                                       

                                                                       

24          Bring up Powerha services on node1                                                  

                      smitty clstart ..  Manually                                                

                                                                       

25          Bring up cluster services on other node                                             

                                                                       

26          Verify  cluster is stable on all nodes                                                    

                          cldump                                              

                          clshowsrv -v                                                    

                                                                       

27          Bring RG online on respective nodes                                                 

                     cd /usr/local/bin/unixops/dr_snaps/pre                                                  

                      cat clRGinfo_v.pre                                             

                                                                       

28          Suspend the application monitoring.                                                 

 

 

                                                         

                                                         

              Prep-Work                       Notes    

                                                         

                                                         

              On all Cluster Nodes                                  

                                                         

A            Pre-requisite  - AIX versions                                     

                             All PowerHA cluster nodes should be at AIX 7.2 TL2 sp2   or AIX 7.2 TL1 sp2  or AIX 7.1 TL4 sp2                         

                                                               oslevel -s       AIX (7200-02-02-1810)   or  AIX (7100-04-02-1614)  or AIX (7200-01-02-1717)            

                             Note:  Do not upgrade PowerHA  when AIX versions are at different level on cluster nodes               

                                                         

                                                               halevel -s       PowerHA 7.1.3 SP5         

                                                         

B            Check for Repository disk -   5 GB SHARED   with "caavg_private" available on both nodes                            

                             lspv | grep -i caa                            

                                               hdiskXX          00ca8cb74aedb591                    caavg_private   active                   

                                                         

C            PowerHA software  copying                                      

                             [nim:                  

                             scp /export/software/PowerHA/PowerHA_72/PowerHA_72_mig.tar  <USERID>@<TARGETserver>:/sysbkup/               

                                                         

D            Snaps                                 

                   a. Take snaps                                           

                             ls -ltr /usr/local/bin/unixops/scripts                     

                                                         

                             Copy the script  copy entire directory from pugsasnim:/usr/local/bin/unixops/scripts to target AIX server    

                             [nim:/sysbkup]$ scp /sysbkup/scripts_unixops.tar USERID@TARGETserver:/var/tmp                         

                             tar -xvf /var/tmp/scripts_unixops.tar                   

                             /usr/local/bin/unixops/scripts/dr_snaps_pre.ksh              run the Pre snap script   

                                                         

                b.  Take PowerHA / GPFS snaps if running                                      

                             refer to  tab "Snaps"                    

                                                         

E            mksysb                             

                 a.   Take mksysb           /usr/local/bin/run_mksysb.ksh               

                 b.   Verify         ls -ltr /sysbkup               

                 c.   Copy mksysb on NIM server             cat /etc/niminfo | grep -w NIM_MASTER_HOSTNAME                      

                                   Note:  copy  mksysb on respective NIM under /vioclntsbkup                 

                                                         

F            Create cluster snapshot                            

                             smitty sysmirror >                         

                                  Cluster Nodes and Networks                

                                     Manage the Cluster              

                                        Snapshot Configuration                  

                                            Create a Snapshot of the Cluster Configuration                

                                                         

                                                         

                             Cluster Snapshot Name                              [snapshot_<date>]                                    

                               Custom-Defined Snapshot Methods                    []                          

                             * Cluster Snapshot Description                       [Snapshot taken on <Date>]                         

                                                         

                             . Check the snapshot in directory on Primary node                         

                                              /usr/es/sbin/cluster/snapshots/                         

                                                         

G            Clone the disk                                

                                                         

              i.  Disk cloning                                

                             alt_disk_copy -d   hdiskX               on both nodes   

                                                         

x            Check "High-level" steps  tab                                    

                             . Use high level steps if migrating PowerHA on both nodes (not rolling migration)                  

                                                         

                                                         

              Implementation -  During Actual Change window                                          

                                                         

                                                         

1            Apps / Database teams to stop  their respective  services                                           

                                                         

2            Bring down Cluster services on both nodes                                      

                                                         

              On both nodes                              

                                                         

              Stop PowerHA on  both nodes                                 

                                                         

                             clshowsrv -v                    

                             clRGinfo -v                       

                             clmgr query cluster | grep -i state            STATE="STABLE"              

                                                         

                             .  Suspend application monitor -               

                                                         

                             . Stop cluster                    

                                 smitty clstop                

                                                 (only select one inactive node)                       

                                                         

                                                                                    [Entry Fields]                     

                             * Stop now, on system restart or both                 now                    

                               Stop Cluster Services on these nodes               [OFFLINE NODE]                               

                               BROADCAST cluster shutdown?                         False                       

                             * Select an Action on Resource Groups                 Bring Resource Groups Offline                            

                             Note: Please ensure none of the Resources are ACTIVE on this node                          

                             tail -f /var/hacmp/log/hacmp.out                          

                                                         

                             lssrc -ls clstrmgrES          Current state: ST_INIT    

                             clshowsrv -v       Make sure Power HA daemons inactive  

                                           Status of the RSCT subsystems used by PowerHA SystemMirror:  

                                           Subsystem         Group            PID          Status       

                                           cthags           cthags           12255618     active      

                                           ctrmc            rsct             7864590      active          

                                                         

                                           Status of the PowerHA SystemMirror subsystems:             

                                           Subsystem         Group            PID          Status       

                                           clstrmgrES       cluster          14418220     active    

                                                         

                                           Status of the CAA subsystems:   

                                           Subsystem         Group            PID          Status       

                                           clcomd           caa              6553926      active        

                                           clconfd          caa              7471544      active         

                                                         

                                           Details of PowerHA SystemMirror cluster manager:             

                                           Current state: ST_INIT    

                                                         

                b. In case of  GPFS     -  check Quorum  and stop GPFS before upgrade                                          

                             ps -ef | grep -i mmfs                    

                             mmgetstate -aLs                            

                             mmshutdown -N <nodename>   Stop GPFS Only on Node where upgrading,  GPFS filesystem should stay up on other nodes           

                             mmgetstate -a                

                             cp -p /etc/inittab /etc/inittab.gpfs                         

                             vi /etc/inittab                   

                             :mmfs:2:once:/usr/lpp/mmfs/bin/mmautoload reboot >/dev/console 2>&1       Comment out GPFS        

                                                         

                             ls -ltr /etc/rc.d/rc2.d | grep -i gpfs                          

                             cd /etc/rc.d/rc2.d                        

                             mv S999gpfs s999gpfs                  

                                                         

              c.  Make sure to disable CRS autostart for DB severs(check with DBA's)                              

                                                         

                                                         

3            Boot with Cloned disk                                

                             Reboot the server after services are down                        

                                                         

4            Validation check  before PowerHA migration                                  

                                                         

              a.  AIX version                                

                             oslevel -s            Example:AIX (7200-02-02-1810)               

                             lppchk -v                          

                                                         

                b.  PowerHA    halevel -s            PowerHA  (7.1.3 SP5)    

                                                         

5            PowerHA software  copying                                      

                             [pugsasnim:                    

                             scp /export/software/PowerHA/PowerHA_72/PowerHA_72_mig.tar  <USERID>@<TARGETserver>:/sysbkup/               

                                                         

6            check "rhosts" file                                        

                             cat /etc/cluster/rhosts  add IP's  of Both nodes and  one line per IP  (Note:  Service IP is optional and not needed)             

                                                         

7            Cluster Daemons                                         

                             lssrc -a | grep -i cl            clcomd           caa              6947012      active   

                                           clconfd          caa              7536862      active          

                                           clstrmgrES       cluster          14418136     active    

                                           clevmgrdES       cluster          22609998     active  

                                           gsclvmd                           23986188     active         

                                           nimsh            nimclient                     inoperative   

                                           secldapclntd     secldap                       inoperative

                                                         

                             ps -ef | grep -i clcomd       root  5439664  3539196   0 10:59:22      -  0:00 /usr/sbin/clcomd -d 

                                                         

                             clshowsrv -v|grep 'Current state'             STATE="INIT"    

                                                         

8            Refresh  "clcomd"                                        

                             lssrc -s clcomd   clcomd           caa                           inoperative        

                             #   startsrc -s clcomd                  (in case inoperative)         clcomd           caa                           Active    

                             refresh -s clcomd            Note:  refresh of "clcomd"  is needed

                                                         

9            Update these files                                      

                             cp -p /etc/netsvc.conf /etc/netsvc.conf.prehamig                          

                             cat /etc/netsvc.conf                    

                                             hosts = local, bind4                     

                             .if sequence is incorrect, then correct it.              

                             vi /etc/netsvc.conf                       

                                       hosts = local, bind4             Note:  Only this line should be in file,  all commented lines should be removed from file        

                                                         

10          caa_cfg  uncomment in /etc/inetd.conf                              

                                                         

                             grep -i caa /etc/inetd.conf                        

                                                                                       caa_cfg stream  tcp6    nowait  root    /usr/sbin/clusterconf clusterconf >>/var/adm/ras/clusterconf.log 2>&1                  

                             .if sequence is incorrect, then correct it.              

                             a. Copy  and uncomment                           

                             #cp -p /etc/inetd.conf   /etc/inetd.conf.old1                     

                             vi /etc/inetd.conf                         

                             uncomment the caa_cfg line                     

                             b. Refresh daemon                       

                             refresh -s inetd               

                                                         

11          PowerHA 7.2 migration                             

                                                         

              Install the new PowerHA systemmirror 7.2.2.1  software on both nodes                                

                                                         

              a.  Software Source directory                                  

                             tar -xvf /sysbkup/PowerHA_72_mig.tar               

                             cd /sysbkup/PowerHA_72_mig                

                                                         

                             ls -ltr                   

                             drwxrwxr-x    2 root     system         4096 May 15 13:50 PowerHA_72_base                       

                             drwxr-xr-x    3 root     system          256 Aug 23 11:15 PowerHA_72_patches                  

                                                         

              x.  Install efix                                 

                             cd /sysbkup/PowerHA_72_mig/PowerHA_72_patches/efix_IJ04268_for_AIX7222              No reboot required for this efix

                             emgr -p -e  IJ04268s2a.180403.AIX72TL02SP02.epkg.Z                  

                             emgr -X -e  IJ04268s2a.180403.AIX72TL02SP02.epkg.Z                   

                                                         

              b.  Commit all other filesets                                     

                             installp -c all      Note:If complians about any gpfs efix, please ignore    

                                                         

                  Verify PowerHA is down                                       

                             clshowsrv -v                    

                             clRGinfo                           

                                                         

              c.  PowerHA 7.2.0.0  -   base code  install   (update from older version)                                           

                             cd /sysbkup/PowerHA_72_mig/PowerHA_72_base                       

                             installp -acgXY -d /sysbkup/PowerHA_72_mig/PowerHA_72_base all                        

                                                         

                 Check                             

                             lppchk -v             should be clean 

                             oslevel -s            Example:7200-02-02-1810          

                                                         

                             lslpp -l | grep -i cluster   cluster.es.server.rte      7.2.2.0  COMMITTED  Base Server Runtime          

                                                         

              d.  PowerHA 7.2.2.1  update                                    

                                                         

                             cd /sysbkup/PowerHA_72_mig/PowerHA_72_patches/PowerHA_7221                      

                             installp -acgXY -d /sysbkup/PowerHA_72_mig/PowerHA_72_patches/PowerHA_7221 all                 

                                                         

                             lslpp -l | grep -i cluster   cluster.es.server.rte      7.2.2.1  APPLIED    Base Server Runtime

              x.  Install efix                                 

                             check for OS version and install efix accordingly oslevel -s             

                For 7200-02-02-1810    cd /sysbkup/PowerHA_72_mig/PowerHA_72_patches/efix_IJ04268_for_AIX7222              No reboot required for this efix

                             emgr -p -e  IJ04268s2a.180403.AIX72TL02SP02.epkg.Z                  

                             emgr -X -e  IJ04268s2a.180403.AIX72TL02SP02.epkg.Z                   

                                                         

              For 7100-04-02-1614                                 

              /sysbkup/PowerHA_72_mig/PowerHA_72_patches/efix_IJ04512_for_AIX7142/              No reboot required for this efix

                             emgr -p -e  IJ04512m2b.180718.AIX71TL04SP02.epkg.Z                

                             emgr -X -e  IJ04512m2b.180718.AIX71TL04SP02.epkg.Z                 

              For 7200-01-02-1717                                 

              /sysbkup/PowerHA_72_mig/PowerHA_72_patches/efix_IJ04267_for_AIX7212/              No reboot required for this efix

                             emgr -p -e IJ04267m2a.180403.AIX72TL01SP02.epkg.Z                  

                             emgr -X -e  IJ04267m2a.180403.AIX72TL01SP02.epkg.Z                 

                                                         

                 Check                             

                             emgr -l  CAA clcomd combo fix   

                             lppchk -v             should be clean 

                             oslevel -s            Example:7200-02-02-1810          

                             halevel -s            7.2.2 SP1            

                                                         

12          Reboot the server                                        

                             . Make sure GPFS is not running  otherwise have to stop it              mmshutdown -N <Nodename> 

                                             ps -ef | grep -i mmfs ; lslpp -l | grep -i gpfs                      

                             . Reboot the server         shutdown -Fr    

                                              shutdown -Fr                 

12a        Check ODM HA version                             

                             odmget HACMPnode | grep -i version                  

                                                   version = 18              (this means one node is updated)

                                                   version = 18              

                                                  version = 15 It will show version 15 once other node is updated

                                                   version = 15              

                                                         

12          Start cluster                                   

                             x. start cluster services one node at a time                        

                             smitty clstart                   

                                                                                     [Entry Fields]                     

                             * Start now, on system restart or both                now                                                                                                                                                  

                               Start Cluster Services on these nodes              [NODE NAME]                                                                                                                                   

                             * Manage Resource Groups                              Automatically              Note: No manual option,start with default automatic option, later in steps it will show manual       

                               BROADCAST message at startup?                       true                                                                                                                                  Note:During intial cluster startup after migration it will show the message but later on after sync it will be fine.    

                               Startup Cluster Information Daemon?                 false                                                                                                                                 Verifying Cluster Configuration Prior to Starting Cluster Services.

                               Ignore verification errors?                         false                                                                                                                                                

                               Automatically correct errors found during           Interactively                                                                                                                         Cluster services are running at different levels across     

                               cluster start?   the cluster.  Verification will not be invoked in this environment.       

                                                         

                             tail -f /var/hacmp/log/hacmp.out                          

                             clRGinfo -v                       

                             clRGinfo -m                     

                             clshowsrv -v                    

                                                         

12a        Validate ODM HA version                                         

                             odmget HACMPnode | grep -i version                  

                                                   version = 18              (this means one node is updated)

                                                   version = 18              

                                                  version = 18 It will show version 15 once other node is updated

                                                  version = 18

                                                         

14          Check                               

                             cldisp | more                  

                                           Cluster: star_cl 

                                              Cluster services: active             

                                              State of cluster: up     

                                                 Substate: stable        

                                                         

                             cllsif      Note: You will see heartbeat disk which will go away aft
