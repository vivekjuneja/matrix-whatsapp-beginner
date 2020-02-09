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
mkdir mautrix-whatsapp
cd mautrix-whatsapp
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
docker run -v /Users/vivekjuneja/Projects/matrix_experiments/mautrix-whatsapp/registration.yaml:/data/registration.yaml:rw  -d --name synapse-server \
    --mount type=volume,src=synapse-data-new,dst=/data \
     -v $(pwd)/homeserver.yaml:/data/homeserver.yaml \
    -p 8008:8008 \
    matrixdotorg/synapse:latest
```

6. Start the WhatsApp bridge
```
docker run --restart unless-stopped -p 29318:29318 -v `pwd`:/data:z dock.mau.dev/tulir/mautrix-whatsapp:latest
```









