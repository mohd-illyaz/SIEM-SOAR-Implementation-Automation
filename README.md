# SOC Automation lab with SOAR

 <p align="center">
Configuration Reference Diagram: <br/>
<img src="https://imgur.com/TT4mect.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
     
## Objectives
- Installation+Configuration of Wazuh & TheHive Servers on the Cloud
- Setting up a Windows 10 client on VirtualBox, installing Sysmon and Wazuh agent in that
- Generating Telemetry via Mimikatz & Ingesting Logs into Wazuh
- Integrateing Shuffle SOAR and employing VirusTotal
- Directing alerts to Shuffle and escalating it to TheHive for case management
- Mailing it to SOC analysts for the investigation then enabling proactive response by the Wazuh to secure Windows.

### Tools Used
- Digitalocean
- Wazuh
- TheHive
- Shuffle
- VirtualBox
- Windows
- Ubuntu
- Sysmon
- Mimikatz
- Virustotal
- Powershell


## Walkthrough

First, we are going to be dealing with a tool called Wazuh. Wazuh is a cybersecurity tool that integrates SIEM + XDR capabilities. Comes with a load of features like Log Data analysis, Incident Response, Intrusion Detection and many more!

Second, we are also going to be using the The Hive in which it’s a 4 in 1 Incident Response tool. We will be using it as our case management tool!

Third, we will have a Windows 10 virtual machine with Sysmon installed.

Sysmon is a valuable tool for enhancing the security posture of Windows-based systems by providing detailed monitoring and logging capabilities essential for Incident Response and more. It will make logging data into Wazuh so much easier once we have our indexer agent installed. It will forward the events generated by Sysmon directly into Wazuh making the data easily visible to analyze.

First, we are going to begin by Installing VirtualBox and Installing Windows 10 into it. Next, we are going to download Sysmon straight from Microsoft’s website. Once, that’s done we are going to begin installing Sysmon .XML file. Once both are finished downloading we extract the zipped Sysmon file right in the Downloads directory. We are going to copy the file path for our next step.

From there we shall open Powershell with administrative privileges. We are then going to go over to run the following commands shown below:

<br/>
<img src= "https://imgur.com/ZcFV1AS.png" height="80%" width="80%" alt=""/>
<br />
<br />

We are going to check our directory to make sure Sysmon executable and the configuration file is in there. Next, we are going to run the executable and we see that it doesn’t run but shows us the help command to guide us how to go about using Sysmon, confirming we don’t have Sysmon installed.

./sysmon64.exe -i sysmonconfig.xml

We may double check by running ‘Services & Event Viewer’.

Click and type ‘S’ scroll down until we see Sysmon
Application and Services>Microsoft>Windows>Click and type ‘S’
Once, we see it in there we are good to go! If, we click on Sysmon in Event Viewer and click on ‘Operational’. We see a ton of telemetry!

<br/>
<img src= "https://imgur.com/5aUgEZa.png" height="80%" width="80%" alt=""/>
<br />
<br />

Now that we have Windows 10 installed with Sysmon we can go ahead and detect malicious activity!

Next, we are going to install Wazuh via Digital ocean. Create a new droplet
We shall then proceed to select an Ubuntu image running on 22.04 LTS x64.
Requirements is at least 8GB ram and 50gb of Disk space 

<br/>
<img src= "https://imgur.com/i3WYCNQ.png" height="80%" width="80%" alt=""/>
<br />
<br />

Now, hover over to the “firewall” tab. Begin, creating our firewall.
Change our Inbound to “All TCP” ports and only specify our IP address to be able to access our droplet.
We can do a look up in our search engine, “What is my IP address”? It should provide it to you just copy and paste it into our firewall configuration.
We will want to do the same for UDP just in case. Scroll down and click on “Create Firewall.” Once our firewall is created we can start adding our virtual machine to it.

<br/>
<img src= "https://imgur.com/eFwFS4b.png" height="80%" width="80%" alt=""/>
<img src= "https://imgur.com/YUXCcmg.png" height="80%" width="80%" alt=""/>
<br />
<br />

On the left hand side, Click on Droplets and Click on our newly created Wazuh droplet.
Inside the droplet let’s click on Networking. Scroll down until you see Firewalls.
Click “Edit” It will show the newly created firewall we just created click on it. Scroll over to our droplet tab and click “Add Droplets.”

<br/>
<img src= "https://imgur.com/TrUP4ZE.png" height="80%" width="80%" alt=""/>
<img src= "https://imgur.com/DlpxGH9.png" height="80%" width="80%" alt=""/>
<br />
<br />

It should’ve successfully been added and should be up to date.
Under, the droplets tab and under Wazuh. Click on the Access tab within the droplet. There will be a droplet console to allow remote access to our Virtual Machine.

<br/>
<img src= "https://imgur.com/vjHIACT.png" height="80%" width="80%" alt=""/>
<br />
<br />

Click “Launch Droplet Console”. Give it time to connect to droplet. Once, we have successfully connected via SSH.

Note: I was having issues with droplet console in DigitalOcean so i decided to SSH into Powershell instead.
ssh root@<IP address of droplet>

Since, we are already in root we can run:

apt-get update && apt-get upgrade -y

<br/>
<img src= "https://imgur.com/B20yQUL.png" height="80%" width="80%" alt=""/>
<br />
<br />

Press enter. Again it will ask you which services you would like to run again hit enter. Once, finished we may begin with the installer of Wazuh.

Which, can be found on Wazuh’s website for reference.

curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a

Give it a couple minutes for Wazuh to install. Once it’s finalized it will display the username and password to access Wazuh dashboard. Make sure to document the credentials within a notepad or password manager for the remainder of the lab.

In order to login into Wazuh we must copy our server’s public IP address. Afterwards within our browser let’s paste in our Server IP via:

https://<server IP>

<br/>
<img src= "https://imgur.com/AI5fhcM.png" height="80%" width="80%" alt=""/>
<br />
<br />

t will prompt you that the connection is insecure. That’s okay click on ‘Advanced’ and click on ‘Proceed to <Server IP>(unsafe)’. It should redirect us to Wazuh’s dashboard. We are in! >:)

Now is to install and configure TheHive. Let’s go back into DigitalOcean and hover over to Create in the top right corner. Click on it.
We will want to click the Server in the region closest to us.
Scroll down to choose an image. We will go with Ubuntu 22.04 LTS x64 like before.

Scroll down again to Sizes and we will go with these allocated resources.

<br/>
<img src= "https://imgur.com/v3ysYLr.png" height="80%" width="80%" alt=""/>
<br />
<br />

Next, let’s make sure we place the “thehive” droplet we just created into the firewall we had created before.
Click on our recently created Firewall.
Click on the Droplets tab. It should show a button to “Add Droplets.” Click on it and search up “thehive” droplet we just created.
Add the droplet to our firewall. Sweet! We should get a prompt we updated our firewall successfully! 
We can verify it by going back to our firewall and our newly created “thehive” droplet should be there.

<br/>
<img src= "https://imgur.com/OU1AUqK.png" height="80%" width="80%" alt=""/>
<br />
<br />

Note: I am going to be using Powershell again to SSH since the DigitalOcean console is giving me issues. Remember to copy the thehives server public IP.

ssh root@<Server IP>

Once, we are in let’s copy and paste TheHive’s prerequisites by running:

apt install wget gnupg apt-transport-https git ca-certificates ca-certificates-java curl software-properties-common python3-pip lsb-release

Once, the prerequisties are done installing we are next going to be installing 4 components: Java, Cassandra, ElasticSearch, lastly TheHive.

Note: Run each command one at a time it will not run the commands together in Powershell.

First up we are going to install Java by running the command below:

wget -qO- https://apt.corretto.aws/corretto.key | sudo gpg –dearmor -o /usr/share/keyrings/corretto.gpg
echo “deb [signed-by=/usr/share/keyrings/corretto.gpg] https://apt.corretto.aws stable main” | sudo tee -a /etc/apt/sources.list.d/corretto.sources.list
sudo apt update
sudo apt install java-common java-11-amazon-corretto-jdk
echo JAVA_HOME=”/usr/lib/jvm/java-11-amazon-corretto” | sudo tee -a /etc/environment
export JAVA_HOME=”/usr/lib/jvm/java-11-amazon-corretto”

Press Enter.

Next, we are going install Cassandra running the following command below:

wget -qO – https://downloads.apache.org/cassandra/KEYS | sudo gpg –dearmor -o /usr/share/keyrings/cassandra-archive.gpg
echo “deb [signed-by=/usr/share/keyrings/cassandra-archive.gpg] https://debian.cassandra.apache.org 40x main” | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
sudo apt update
sudo apt install cassandra

Third, install ElasticSearch by running the following command shown below:

wget -O- https://archives.strangebee.com/keys/strangebee.gpg | sudo gpg –dearmor -o /usr/share/keyrings/strangebee-archive-keyring.gpg
echo ‘deb [signed-by=/usr/share/keyrings/strangebee-archive-keyring.gpg] https://deb.strangebee.com thehive-5.2 main’ | sudo tee -a /etc/apt/sources.list.d/strangebee.list
sudo apt-get update
sudo apt-get install -y thehive

After, they have all successfully installed. We shall go ahead and configure them so they may work together seamlessly.

First, we will want to configure Cassandra in “thehive” server

nano /etc/cassandra/cassandra.yaml

Replace the ‘clustername’ with a name of your choosing in my case, it’s ‘eddythir’.

ctrl+w and search listen_address

Change it to our ‘thehives’ public IP address. Do the same for rpc_address.

ctrl+w and search rpc_address

Change it to our ‘thehives’ public IP address again. Do the same for seed_provider.

ctrl+w and search seed_provider

Change the default address with our servers public IP address. By also, keeping the default port :7000. Save it by using;

ctrl+x and it will prompt you to overwrite the file type y for yes

steps are mentioned clearly in the following images:

<br/>
<img src= "https://imgur.com/oSYTevE.png" height="80%" width="80%" alt=""/>
<img src= "https://imgur.com/yNMaenT.png" height="80%" width="80%" alt=""/>
<img src= "https://imgur.com/sLFtJEP.png" height="80%" width="80%" alt=""/>
<br />
<br />

Now we are going to stop the Cassandra service with first command shown below. Second, we are going to delete all files in the /var/lib/cassandra/ directory. Third, we are going to restart the service using the third command. Last, its a good habit to check if our desired Cassandra service is running by using the last command shown below. We see that it’s active and running.

<br/>
<img src= "https://imgur.com/CSgJxdL.png" height="80%" width="80%" alt=""/>
<br />
<br />

Next, we are going to configure the configuration files for ElasticSearch.

nano /etc/elasticsearch/elasticsearch.yml

Once, in scroll down and remove the comment on ‘cluster.name’.

Let’s change it to “thehive.”

Scroll down again and remove the comment on ‘node.name’.

We shall leave it as ‘node-1’.

Scroll down even more and remove the comment on ‘network.host’.

Delete the current host IP and input thehive’s public IP address.

Scroll down a bit after that and remove the comment on ‘http.port:9200’.

Scroll down a bit more and remove the comment on ‘cluster.initial_master_nodes:’.

Remove “node-2” and just leave “node-1”.

cluster.initial_master_nodes: [“node-1”]

It should look like this. We don’t need to scroll any further because there is no need to scale in a demo environment. Let’s save it.

Next, let’s start by starting the elasticsearch service.

systemctl start elasticsearch

Next, enable it by running the following:

systemctl enable elasticsearch

Lastly, we will want to check the status of elasticsearch to verify it’s running.

<br/>
<img src= "https://imgur.com/Hige1WI.png" height="80%" width="80%" alt=""/>
<br />
<br />

Next, we will want to get into configuring the configuration files for TheHive. We want to make sure a specific user and group have access to a certain file path.

Note: Remember our Powershell command didn’t install TheHive in the beginning. We must manually reinstall TheHive.

We will then ls -la into the /opt/thp directory. It will show us file permissions for the root user. 
We will then want to change those permissions to user & group ‘thehive’ recursively within the /opt/thp directory.
We shall then proceed to nano into applicaiton.conf file to change it’s configurations. Scroll down a bit until you see to:
Change both hostnames shown above to our ‘thehive’ public IP address. Also, change cluster-name to the one previously.

<br/>
<img src= "https://imgur.com/8VLbw3p.png" height="80%" width="80%" alt=""/>
<img src= "https://imgur.com/etrvrT4.png" height="80%" width="80%" alt=""/>
<br />
<br />

Scroll down a bit more.
In ‘application.baseUrl = “http://localhost:9000” to ‘thehive’ public IP address as shown above.

Also, I see that the Cortex and MISP are enabled by default but, can be disabled. Cortex is their data enrichment and response capability. While, MISP is used for CTI (Cyber Threat Intelligence) platform.

Let’s go ahead and save the configuration file.


<br/>
<img src= "https://imgur.com/Sd72fyU.png" height="80%" width="80%" alt=""/>
<br />
<br />

Start and enable the hive using the commands below. Then, verify if TheHive is actually running. It’s good habit to check the other services too.

<br/>
<img src= "https://imgur.com/4n6W1AT.png" height="80%" width="80%" alt=""/>
<br />
<br />

NOW DO,
http://<Your Server IP>:9000

user: admin@thehive.local

password: *******

<br/>
<img src= "https://imgur.com/QNpgOue.png" height="80%" width="80%" alt=""/>
<br />
<br />

We are in! Now, that we have TheHive configured we will move onto configuring Wazuh server.

Let’s login using our credentials we had saved.

We will immediately met with no agents installed into our manager.
So Let’s hop right back into Wazuh server terminal and run the ls command to check if the Wazuh install files exist. We then want to extract .tar Wazuh install file. Right afterwards want to change right into that directory.
Go right back into that directory and ls into it to see what we’ve got. We are interested in the “wazuh-passwords.txt” file. We shall concatenate into the file.

<br/>
<img src= "https://imgur.com/z82uEDe.png" height="80%" width="80%" alt=""/>
<img src= "https://imgur.com/KGkSPKw.png" height="80%" width="80%" alt=""/>
<br />
<br />

We are going to be using the Wazuh API user credentials later on in the lab.

<br/>
<img src= "https://imgur.com/uK53yK3.png" height="80%" width="80%" alt=""/>
<br />
<br />

Let’s get back into Wazuh and add the missing agent by clicking ‘Add agent’ button.

Scroll down and click on Windows.

<br/>
<img src= "https://imgur.com/xSAPRZa.png" height="80%" width="80%" alt=""/>
<br />
<br />

Copy and paste this command once your configurations are ready.

net start wazuhsvc

To begin starting the Wazuh service. Additionally, we can start the Service in the “Services” program.

Back to Wazuh we should see that our Agent is connected. We just successfully configured our Wazuh server 

<br/>
<img src= "https://imgur.com/0QxM1WO.png" height="80%" width="80%" alt=""/>
<br />
<br />

We can maneuver into Security Events and begin querying Security Events in Wazuh!!! We just configured Wazuh & TheHive and now working as expected,

Now on to the next stage ;
Generating Telemetry via Mimikatz & Ingesting Logs into Wazuh

Let’s start off by starting up our Windows 10 machine. Next, the file path:

C:\Program Files (x86)\ossec-agent\ossec.conf

It holds all configurations for Wazuh in this .conf file.

Note: Run as administrator and open up notepad for file permissions

<br/>
<img src= "https://imgur.com/FirMjnQ.png" height="80%" width="80%" alt=""/>
<br />
<br />

Scroll down a bit until we get to the log analysis syntax. 
In the local files syntax, do you see that it excludes using the ‘!=’ operator excludes the highlighted event IDs. (!= ‘does not equal to)
Two ways to go about this by either using Sysmon or enabling EventID 4688. we will use Sysmon. In order to do so, we will configure the ossec.conf file and edit it to ingest our Sysmon logs.

<br/>
<img src= "https://imgur.com/JyCuNVn.png" height="80%" width="80%" alt=""/>
<br />
<br />

Let’s make a backup of it just in case we break things.
Back in the configuration file we shall want to copy and paste the first syntax right under local files.

<br/>
<img src= "https://imgur.com/WjlJNKk.png" height="80%" width="80%" alt=""/>
<br />
<br />

We want to change the file path of the Application to Sysmon. We are going to find that specific file path in Sysmon. By following this path:
Application & Services Logs> Microsoft>Windows>Click the services & type ‘S’>Scroll down until we find Sysmon>Double Click it>Right Click Operational>Click Properties

Go back to the configuration file and paste the highlighted text shown above and replace “Application”.

<br/>
<img src= "https://imgur.com/8OMNV6m.png" height="80%" width="80%" alt=""/>
<br />
<br />

It should look exactly as shown above. We can run the same process for Powershell if we wanted to ingest the logs.
We are going to be more thorough and remove the local file application, security, and system event log ingestions.
We could leave it but it will take a lot more resources to ingest and all we need to prioritize are the Sysmon and active-response event logs.

<br/>
<img src= "https://imgur.com/NJ1AQfR.png" height="80%" width="80%" alt=""/>
<br />
<br />

Go ahead and overwrite the ossec.conf file.
Search in the search bar “Services”. Click on the services and type ‘W’. Scroll down until we find Wazuh and Restart the service.

Note: Anytime changing the configurations of a service we must restart the service to apply changes.

Head back over to our Wazuh dashboard and click under Events. Search for “Sysmon.” It might take some time for Sysmon Events to ingest and it’s fine.
Next thing we want to do is download Mimikatz onto our Windows 10 machine. First, we will want to do is disable Defender to allow Mimikatz to execute.

Windows Security>Virus & Threat Protection>Scroll down a bit Managed Settings>Scroll down and click on “Add or remove exclusion”>Click on “Add an exclusion”>Click on “Folder”>Select our “Downloads” folder to exclude

<br/>
<img src= "https://imgur.com/UAFou7a.png" height="80%" width="80%" alt=""/>
<br />
<br />

This will allow us to download Mimikatz within the Downloads folder.
Let’s copy our file path that leads to Mimikatz into Powershell.

<br/>
<img src= "https://imgur.com/fJtjbKt.png" height="80%" width="80%" alt=""/>
<br />
<br />

Open up Powershell and run as administrator. Then, change directories into the file path copied shown below.
Afterwards run Mimikatz as shown below. We will then check Wazuh if there were any events related to Mimikatz.

<br/>
<img src= "https://imgur.com/ogO1G2q.png" height="80%" width="80%" alt=""/>
<br />
<br />

We get sysmon logs but not Mimikatz. By default Wazuh doesn’t detect everything. This is where the magic happens we must create detection rules for it, to look for Mimikatz specifically. It can be changed by going into the Wazuh manger and configuring the ossec.conf file to force it to look at everthing or create rules that looks at specific events like Mimikatz! Once it’s executed it will trigger automatically upon detection. Soon thereafter, we should have the log ingested and investigated.

<br/>
<img src= "https://imgur.com/A73n2a2.png" height="80%" width="80%" alt=""/>
<br />
<br />

By going into our Wazuh SSH terminal from there able to modify the ossec.conf file by forcing it to detect everything.

First, things first we want to create a backup to the ossec.conf file just in case we break things by running the following command
Afterwards, we are going to edit the file using nano.
<br/>
<img src= "https://imgur.com/mZYu2Js.png" height="80%" width="80%" alt=""/>
<br />
<br />

Scroll down a bit and see that <logall> and <logall_json> syntaxes are set to ‘no’. Let’s change both to ‘yes’.
Now, that the configurations have been made go ahead and overwrite the file.
What this does is force Wazuh to archive all logs into a file named “Archive.”
Change directory into the archives directory and ls into it to see the files contained in there. There will be all the logs archived there. What we would like is for Wazuh to ingest those logs.
<br/>
<img src= "https://imgur.com/w4TuoGd.png" height="80%" width="80%" alt=""/>
<img src= "https://imgur.com/xCB4Co1.png" height="80%" width="80%" alt=""/>
<img src= "https://imgur.com/8IVspPj.png" height="80%" width="80%" alt=""/>
<br />
<br />

 Now we will be editing the .yaml file using nano and using the /etc/filebeat/filebeat.yml filepath to access it. Scrolll until you see…
 Changing all filebeat.modules to ‘true’. Afterwards, overwrite the file. Remember, everytime we must restart the service to apply changes.

 <br/>
<img src= "https://imgur.com/3DVJxUz.png" height="80%" width="80%" alt=""/>
<img src= "https://imgur.com/ImGMgwj.png" height="80%" width="80%" alt=""/>
<br />
<br />

Back to Wazuh on the left handside click on Management and right under ‘Stack Management’.

Click on Index patterns and on “Create index pattern.”

Name the Index pattern as shown below and click Next.

Scroll down and click on Timestamp. Afterwards click “Create index pattern.”

Go back on the left handside and click on “Discover.”

Select our ‘archives’ index pattern.

following are the image description of the above mentioned steps:
<br/>
<img src= "https://imgur.com/vpwe3cs.png" height="80%" width="80%" alt=""/>
<img src= "https://imgur.com/reOsUWf.png" height="80%" width="80%" alt=""/>
<img src= "https://imgur.com/0aqxMnp.png" height="80%" width="80%" alt=""/>
<img src= "https://imgur.com/s1w0GDe.png" height="80%" width="80%" alt=""/>
<img src= "https://imgur.com/mBPtwhR.png" height="80%" width="80%" alt=""/>
<img src= "https://imgur.com/Yq1AQdQ.png" height="80%" width="80%" alt=""/>
<br />
<br />

Now give it some time to ingest the logs in regards to Mimikatz.

<br/>
<img src= "https://imgur.com/D0yZn7t.png" height="80%" width="80%" alt=""/>
<br />
<br />

Back in our Wazuh Server CLI we want to follow the directory path using the ‘pwd’ command. Let’s ‘ls’ and see what directories we have in there.
We see that theres an archives.json and archives.log files within the 2024 directory. There are definitely tons of events in those files because of the amount of bytes within those archive files.

<br/>
<img src= "https://imgur.com/wosbPcv.png" height="80%" width="80%" alt=""/>
<br />
<br />




















