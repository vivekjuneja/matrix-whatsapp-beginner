# matrix-whatsapp-beginner
A recipe for setting up Matrix synapse and WhatsApp bridge to allow Riot to talk to WhatsApp

**Setup the Synapse server**

1. Generate the homeserver configuration
```
docker run -it --rm     --mount type=volume,src=synapse-data-new,dst=/data  -e SYNAPSE_SERVER_NAME=$HOST    -e SYNAPSE_REPORT_STATS=yes    matrixdotorg/synapse:latest generate
```

2. Start the Synapse server
```
docker run -d --name synapse-server \
    --mount type=volume,src=synapse-data,dst=/data \
    -p 8008:8008 \
    matrixdotorg/synapse:latest
```

3. Extract the homeserver.yaml from the running Synapse container
```
docker cp synapse-server:/data/homeserver.yaml .
```

4. Enable new user registration in the homeserver.yaml
```
enable_registration: true
```

5. Restart the Synapse container

```
docker stop synapse-server
docker rm synapse-server
docker run -d --name synapse-server \
    --mount type=volume,src=synapse-data-new,dst=/data \
     -v $(pwd)/homeserver.yaml:/data/homeserver.yaml \
    -p 8008:8008 \
    matrixdotorg/synapse:latest
```

6. Verify if the Synapse is running by visiting http://localhost:8008/


**Setup the WhatsApp bridge**

(*Some of the instructions were borrowed from https://github.com/tulir/mautrix-whatsapp/wiki*)

1. Create a directory to host WhatsApp bridge files
```
mkdir $(pwd)/mautrix-whatsapp
cd $(pwd)/mautrix-whatsapp
```

2. Start the bridge container to get the config files
```
docker run --rm -v `pwd`:/data:z dock.mau.dev/tulir/mautrix-whatsapp:latest
```

3. Setup the the homeserver and appservice address in the config.yaml. The <HOSTNAME> points to your hostname. 
```
homeserver:
    # The address that this appservice can use to connect to the homeserver.
    address: http://<HOSTNAME>:8008
    # The domain of the homeserver (for MXIDs, etc).
    domain: <HOSTNAME>

appservice:
    # The address that the homeserver can use to connect to this appservice.
    address: http://localhost:29318
    # The hostname and port where this appservice should listen.
    hostname: 0.0.0.0
    port: 29318
```
3. Generate the App service registration

``` 
docker run --rm -v `pwd`:/data:z dock.mau.dev/tulir/mautrix-whatsapp:latest
```

4. Configurate the app service config file in the Synapse homeserver configuration
```
app_service_config_files:
  - /data/registration.yaml
```

5. Also add the permission in the config.yaml of the WhatsApp bridge. <HOSTNAME> below should point to your hostname
```
Add permission in the config.yaml 
  permissions:
    '*': 5
    '@admin:example.com': 100
    example.com: 10
    <HOSTNAME>: 10
```

5. Start the Synapse server by mounting the registration.yaml to the Synapse container with read-write permission. 
```
docker stop synapse-server
docker rm synapse-server
docker run -v $(pwd)/mautrix-whatsapp/registration.yaml:/data/registration.yaml:rw  -d --name synapse-server \
    --mount type=volume,src=synapse-data-new,dst=/data \
     -v $(pwd)/homeserver.yaml:/data/homeserver.yaml \
    -p 8008:8008 \
    matrixdotorg/synapse:latest
```

6. Start the WhatsApp bridge
```
docker run --restart unless-stopped -p 29318:29318 -v `pwd`:/data:z dock.mau.dev/tulir/mautrix-whatsapp:latest
```

7. Check if the WhatsApp bridge is working properly by checking the logs 
```
[Feb  9, 2020 23:34:28] [DEBUG] Initializing database
[Feb  9, 2020 23:34:28] [DEBUG] Initializing state store
[Feb  9, 2020 23:34:28] [DEBUG] Initializing Matrix event processor
[Feb  9, 2020 23:34:28] [DEBUG] Initializing Matrix event handler
[Feb  9, 2020 23:34:28] [INFO] Bridge initialization complete, starting...
[Feb  9, 2020 23:34:28] [Database/Upgrade/INFO] Database currently on v12, latest: v12
[Feb  9, 2020 23:34:28] [DEBUG] Checking connection to homeserver
[Feb  9, 2020 23:34:28] [DEBUG] Starting application service HTTP server
[Feb  9, 2020 23:34:28] [DEBUG] Starting event processor
[Feb  9, 2020 23:34:28] [INFO] Bridge started!
[Feb  9, 2020 23:34:28] [Matrix/INFO] Listening on 0.0.0.0:29318
[Feb  9, 2020 23:34:28] [DEBUG] Updating bot profile
[Feb  9, 2020 23:34:28] [DEBUG] Starting users
[Feb  9, 2020 23:34:28] [DEBUG] Starting custom puppets
```


**Use a Matrix client to test the setup**

0. I am using the Riot client for this setup 

1. Create an account from the Riot client on the Synapse using the Identity server as the local machine

![](image5.png?raw=true)


2. Add the WhatsApp bridge as a new people

![](image4.png?raw=true)

3. WhatsApp bridge bot is added 

![](image3.png?raw=true)

4. Login to WhatsApp using WhatsApp Web. Issue a login command to the WhatsApp bridge bot. Scan the QR code using your WhatsApp client on phone using the WhatsApp web mode. 

![](image2.png?raw=true)

5. Synapse will download all contacts and messages. Pick up an existing contact and send a messge

![](image1.png?raw=true)

**And Viola, everything should have worked so far. Welcome to Matrix and WhatsApp bridge**
5. 




