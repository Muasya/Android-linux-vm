# Android-linux-vm
A linux vm to run android container on the cloud and serve the UI on the browser.


#Set up the AAOS Emulator service

To set up the Emulator service:

    Install the Android Emulator Docker container script:

```git clone https://github.com/google/android-emulator-container-scripts.git```

```cd android-emulator-container-script```
```source ./configure.sh```

This activates a virtual environment and makes the executable emu-docker available. To get detailed information about its use, launch it:

```emu-docker -h```

To create the Docker containers, accept the license agreements.
Build the AAOS Emulator Docker container.
Download an emulator build later than version 7154743. For example:

```sdk-repo-linux-emulator-7154743.zip```

Download the AAOS emulator system image. For example, sdk-repo-linux-system-images-7115454.zip:

```emu-docker create <emulator-zip> <system-image-zip>```

Create the Web Containers and set username and password for remote access.

```./create_web_container.sh -p user1,passwd1```

Start the AAOS Emulator Web Service:

```docker-compose -f js/docker/docker-compose-build.yaml -f js/docker/development.yaml up```

You've successfully started an AAOS Emulator Web Service! Use the following to access it on a web browser:

```https://<VM_External__IP>```

#Troubleshooting

If a connection error to the VM external IP occurs, make sure the VM is set up to allow both HTTP and HTTPS traffic. To validate this, see Running a basic Apache web server.
Set up the turn server

You can always use your own turn server. Provided below is an sample on a Google Cloud VM instance.

Note: To make the turn server work on a Google Cloud VM instance, be sure to configure the VM firewall rule to allow traffic on TCP and UDP ports 3478 and 3479.

    Install the coturn server:

```
sudo apt install coturn
systemctl stop coturn
echo "TURNSERVER_ENABLED=1"|sudo tee -a /etc/default/coturn
```

Modify ```/etc/turnserver.conf``` by adding the following lines:

```lt-cred-mech```
#set your realm name
```realm=test```
#coturn username and password
```user=test:test123```
#external-ip=<VM-Public-IP>/<VM-Private-IP>
```
external-ip=34.193.52.134/10.128.0.2

systemctl start coturn
```

Modify the Docker Compose YAML file to include the TURN configuration:

```
cd android-emulator-container-script
nano  js/docker/docker-compose-build.yaml
```

Add the following two environment lines in the emulator section:

```    shm_size: 128M
     expose:
       - "8554"
+    environment:
+       - TURN=printf $SNIPPET
```

Restart the AAOS Emulator service with the turn configuration. Be sure to replace the turn server IP, username, and credential below with your own:

```
export SNIPPET="{\"iceServers\":[{\"urls\":\"turn:35.193.52.134:3478\",\"username\":\"test\",\"credential\":\"test123\"}]}"
docker-compose -f js/docker/docker-compose-build.yaml up
```