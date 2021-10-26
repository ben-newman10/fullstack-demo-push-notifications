demo for use of WebSockets 
the idea is to allow push notifications to be sent from Elastic to the middleware (Python: FastAPI) and then on up to the UI (JavaScript: Svelte), in order to avoid the UI having to constantly poll the middleware which then has to constantly poll Elastic in response (or perform some kind of caching)

progress so far:
* UI opens a WS to the middleware, and subscribes to messages coming from it
* UI sends a boolean down the WS to the middleware to toggle on/off each channel individually
* MetricBeats is running, populating Elastic with system stats
* multiple parallel independent components in the UI (each subscribed to their own backend data feed)
* installed an Elastic plugin which pushes change notifications out on a websocket - requires Elastic and Kibana 6.5.3, so installed them also (instead of latest version 7.15.1) 
* installed the plugin into Elastic (and then restarted Elastic) which can't be done easily with Elastic running inside a Docker container (so actually installed Elastic and Kibana)
* used default config for the plugin, to push all updates out to the WS, seems to be fine as-is (for this demo at least)
* changed the Python code to use the WS (instead of polling Elastic)
* totally removed the logic (to determine when the backend status has changed) as the Elastic plugin does this for us now
* totally removed all polling of Elastic from the middleware, and all polling of the middleware from the UI


to start the UI:
    npm run dev


to start Python server:
    cd src; uvicorn main:app


to install *** VERSION: 6.5.3 *** of Elastic, Kibana, Metricbeat
    echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee –a /etc/apt/sources.list.d/elastic-6.x.list
    apt update
    apt install elasticsearch=6.5.3
    apt install kibana=6.5.3
    cd /tmp
    wget https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-6.5.3-amd64.deb
    dpkg -i /tmp/metricbeat-6.5.3-amd64.deb


to install the plugin
    git clone https://github.com/ForgeRock/es-change-feed-plugin.git
    cd es-change-feed-plugin
    mvn clean install  
    mkdir /tmp/elasticsearch
    cp target/es-changes-feed-plugin.zip /tmp/elasticsearch
    cd /tmp/elasticsearch
    unzip es-changes-feed-plugin.zip 
    cd ..
    zip -r es-changes-feed-plugin.zip elasticsearch
    # requires Elastic to be running first
    /usr/share/elasticsearch/bin/elasticsearch-plugin install file:///tmp/es-changes-feed-plugin.zip
    systemctl restart elastic 


to run the demo:
    
    start Python server:
        cd src; uvicorn main:app

    start the UI:
        npm run dev

    start Elastic:
        systemctl start elastic

    start Kibana:
        # requires Elastic to be running first
        systemctl start kibana

    start  Metricbeat:
        # requires Elastic and Kibana to be running first
        /usr/bin/metricbeat setup -e
        systemctl start metricbeat
