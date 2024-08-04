# IoT Infrastructure (v2)

Recommended development environment is Linux (or WSL)

## Container deployment via Azure Container Registry
1. Create an Azure VM with Ubuntu 20.04

2. Add SSH configuration to access from computer (see Confluence)

3. Add network inbound/outbound rules in the Azure web interface (select VM -> Networking -> Add Inbound Rule): 5683/udp

4. Installations on the VM
    * Install Docker (https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04)
    * Install Docker-Compose (https://www.digitalocean.com/community/tutorials/   how-to-install-and-use-docker-compose-on-ubuntu-20-04-de)
       *Note*: newer versions require a '**v**' in front of the version (e.g. v2.6.0 instead of just 1.29.2)
    * Install Azure CLI (https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt&   view=azure-cli-latest)
    * Install Git
   
5. Make sure to make IP-address static 

6. Set environment variables for the application

7. Login to Azure Container Registry (Credentials in KeePass under **Azure Container Registry**)
   ```sh 
   sudo docker login horeichcontainers.azurecr.io 
   ```

8. Make applications ready for deployment
    * Make sure the appsettings.json files are correct
    * Check NLog configuration files
       * Debug: console - Trace, logfile - Trace, insights - Trace (disable)
       * Production: logfile - Info, logfile - Info, i
       * Correct fileName for logFile (*"/var/log/{appName}/log_${shortdate}.json"*)
    * Check Dockerfile

3. Deployment using Azure Container Registry
* (https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-docker-cli?tabs=azure-cli)
* Build container image and push it to the registry
  ```sh
  $ sudo docker build -t {name} . // (e.g. use horeichcontaizers.azurecr.io/iotbridge-v2)
  $ sudo docker images -a
  $ sudo docker login horeichcontainers.azurecr.io // login
  $ sudo docker tag {name} horeichcontainers.azurecr.io/{name} // tagging with login server name (not necessary if build with correct name before)
  $ sudo docker push horeichcontainers.azurecr.io/{name} 
  ```
* If already tagged just use
  ```sh
  $ sudo docker build -t horeichcontainers.azurecr.io/{name} .
  $ sudo docker push horeichcontainers.azurecr.io/{name}
  ```

11. Pull images on VM
  ```sh
  $ sudo docker pull horeichcontainers.azurecr.io/{name}
  ```
* Set environment variables in docker-compose file (ASPNETCORE_ENVIRONMENT=Production)
* For quick build an running container
  ```sh
  $ sudo docker build -t mqttproxyserverimg -f Dockerfile .
  $ sudo docker run -v /etc/ssl/certs:/etc/ssl/certs -v /var/log/mqttproxyserver:/var/log/mqttproxyserver  -e "ASPNETCORE_ENVIRONMENT=Production" -p 8883:8883 --name mqttproxyservercontainer mqttproxyserverimg
  ```
* If we want console output use ASPNETCORE_ENVIRONMENT=Development

10. Run Commands on VM

cd IoTBackend
git pull origin master

sudo docker network prune
sudo docker-compose pull

export PCS_KEYVAULT_NAME=""
export PCS_AAD_APPID=""
export PCS_AAD_APPSECRET=""
export ASPNETCORE_ENVIRONMENT="Release"

sudo -E docker-compose up -d
sudo docker push horeichcontainers.azurecr.io/iotendpoint:latest
sudo docker push horeichcontainers.azurecr.io/iotlink:latest 


4. Test MQTT server
  ```sh
  mosquitto_pub -h 20.224.4.13 -p 8883 -u "ESTWIoTServer" -P "2VosUTxRN6nQ" -t test/topic -m "{\"value1\":20,\"value2\":40}" -i "mosquitto" -d --cafile /mnt/c/WorkDir/ESTWIoTBridge/MqttProxyServer/WebService/Certificates/Release/estw.root.crt --tls-version tlsv1.2 --insecure
  ```
  * the --insecure makes sure we can use the ip-address (here: 20.224.4.13) instead of the hostname (here: horeich-4), read here (http://www.steves-internet-guide.com/mosquitto-tls/), see here how to use DNS to be able to use hostname (https://community.home-assistant.io/t/mqtt-host-name-verification-failure-ssl/337356/4)

5. Subscribe to sensor topic to test JSON data
   ```sh
   mosquitto_sub -h 20.224.4.13 -p 8883 -u "ESTWIoTServer" -P "2VosUTxRN6nQ" -t estw/WDM331/ESTWWaterSense1 -i "mosquitto" -d --cafile /mnt/c/WorkDir/ESTWIoTBridge/MqttProxyServer/WebService/Certificates/Release/estw.root.crt --tls-version tlsv1.2 --insecure
   ```

* Make sure docker images are restart if shutdown

* The folder *.ssh* in *home* directory is hidden

    environment:
      - PCS_KEYVAULT_NAME
      - PCS_AAD_APPID
      - PCS_AAD_APPSECRET
