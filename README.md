# Instructions

## Prerequisites

### Install exporters on validator node
First of all you will have to install exporters on validator node. For that you can use one-liner below
```
wget -O install_exporters.sh https://raw.githubusercontent.com/kj89/cosmos_node_monitoring/master/install_exporters.sh && chmod +x install_exporters.sh && ./install_exporters.sh
```
make sure following ports are open:
- `9100` (node-exporter)
- `9300` (cosmos-exporter)

## Deployment

### System requirements
Ubuntu 20.04 / 1 VCPU / 2 GB RAM / 20 GB SSD

### Install monitoring stack
To install monitirng stack you can use one-liner below
```
wget -O install_monitoring.sh https://raw.githubusercontent.com/kj89/cosmos_node_monitoring/master/install_monitoring.sh && chmod +x install_monitoring.sh && ./install_monitoring.sh
```

### Copy _.env.example_ into _.env_
```
cp $HOME/cosmos_node_monitoring/config/.env.example $HOME/cosmos_node_monitoring/config/.env
```

### Update values in _.env_ file
```
vim $HOME/cosmos_node_monitoring/config/.env
```

| KEY | VALUE |
|---------------|-------------|
| VALIDATOR_IP | Public ip address of you validator |
| VALOPER_ADDRESS | Operator address of your validator, for example, _"agoricvaloper1zyyz4m9ytdf60fn9yaafx7uy7h463n7alv2ete"_ |
| WALLET_ADDRESS | Your validator self-deligate wallet address, for example, _"agoric1zyyz4m9ytdf60fn9yaafx7uy7h463n7a05eshc"_ |
| TELEGRAM_ADMIN | Your user id you can get from [@userinfobot](https://t.me/userinfobot). The bot will only reply to messages sent from the user. All other messages are dropped and logged on the bot's console |
| TELEGRAM_TOKEN | Your telegram bot access token you can get from [@botfather](https://telegram.me/botfather). To generate new token just follow a few simple steps described [here](https://core.telegram.org/bots#6-botfather) |

### Export _.env_ file values into _.bash_profile_
```
echo "export $(xargs < $HOME/cosmos_node_monitoring/config/.env)" > $HOME/.bash_profile
source $HOME/.bash_profile
```

### Update _prometheus_ configuration file
```
sed -i "s/VALIDATOR_IP/$VALIDATOR_IP/g" $HOME/cosmos_node_monitoring/prometheus/prometheus.yml
sed -i "s/VALOPER_ADDRESS/$VALOPER_ADDRESS/g" $HOME/cosmos_node_monitoring/prometheus/prometheus.yml
sed -i "s/WALLET_ADDRESS/$WALLET_ADDRESS/g" $HOME/cosmos_node_monitoring/prometheus/prometheus.yml
```

### Run docker compose
Deploy the monitoring stack
```
cd $HOME/cosmos_node_monitoring
docker compose up -d
```

## Configuration

### Configure Grafana
1. Open Grafana in your web browser. It should be available on port `9999`

![image](https://user-images.githubusercontent.com/50621007/160622455-09af4fbf-2efb-4afb-a8f8-57a2b247f705.png)

2. Login using defaults `admin/admin` and change password

3. Import custom dashboard

3.1. Press "+" icon on the left panel and then choose **"Import"**

![image](https://user-images.githubusercontent.com/50621007/160622732-aa9fe887-823c-4586-9fad-4c2c7fdf5011.png)

3.2. Input grafana.com dashboard id `15991` and press **"Load"**

![image](https://user-images.githubusercontent.com/50621007/160625753-b9f11287-a3ba-4529-96f9-7c9113c6df3a.png)

3.3. Select Prometheus data source and press **"Import"**

![image](https://user-images.githubusercontent.com/50621007/160623287-0340acf8-2d30-47e7-8a3a-56295bea8a15.png)

4. Change your chain explorer url

4.1. Edit **"Top validators missing blocks panel"**

![image](https://user-images.githubusercontent.com/50621007/160623476-50d8bf62-03cd-4de6-92de-53bc2df830cc.png)

4.2. Go to **"Overrides"** and edit **"Data links"**

![image](https://user-images.githubusercontent.com/50621007/160623555-ae7e9d54-9a0b-4ec9-9b1d-278fafe06682.png)

4.3 Change url to your chain data explorer and hit **"Save"**

![image](https://user-images.githubusercontent.com/50621007/160623647-1f23a1dc-35b0-494f-8ba4-fb4d90f1b0c5.png)

4.4. Hit **"Save"** button on the left top corner to save changes to dashboard

5. Congratulations you have successfully configured Cosmos Validator Dashboard


### Configrure Telegram alerting
1. Open conversation with your Telegram bot you created with [@botfather](https://telegram.me/botfather) and type `/start` to activate bot

![image](https://user-images.githubusercontent.com/50621007/160623782-e18a42c4-659d-477b-9189-43d9027d518c.png)

2. Now you are all set! If you want see other commands type `/help`

> If you want learn more about `alermanager-bot` please visit [their github repo](https://github.com/metalmatze/alertmanager-bot/)

## Testing

### Test alerts
1. For simple test you can stop `node-exporter` service for 5 minutes. It should trigger alert
```
systemctl stop node_exporter
```
2. You will see message from bot firing

![image](https://user-images.githubusercontent.com/50621007/161050843-889edc5e-4e27-4778-9010-b9e9e861cc74.png)

3. Now you can start `node-exporter` service back
```
systemctl start node_exporter
```
4. You will get confirmation from bot that issue is resolved

![image](https://user-images.githubusercontent.com/50621007/161051501-6e87cbb1-6699-4557-81ed-9564db57a76f.png)

## Dashboard contents
Grafana dashboard is devided into 4 sections:
- **Validator health** - main stats for validator health. connected peers and missed blocks

![image](https://user-images.githubusercontent.com/50621007/160629676-bc3c4f0f-66df-4a5f-9844-dca308072e7a.png)

- **Chain health** - summary of chain health stats and list of top validators missing blocks

![image](https://user-images.githubusercontent.com/50621007/160629937-52253f35-8782-4dd2-80cc-ad31d0231a84.png)

- **Validator stats** - information about validator such as rank, bounded tokens, comission, delegations and rewards

![image](https://user-images.githubusercontent.com/50621007/160630119-0abad099-b138-4f61-9e73-49506c2295ff.png)

- **Hardware health** - system hardware metrics. cpu, ram, network usage

![image](https://user-images.githubusercontent.com/50621007/160630213-5e92b3ce-92c9-4f48-8856-383ca884b621.png)

## Cleanup all container data
```
cd $HOME/cosmos_node_monitoring
docker compose down
docker volume prune -f
```

## Reference list
Resources I used in this project:
- Grafana Validator stats [Cosmos Validator by freak12techno](https://grafana.com/grafana/dashboards/14914)
- Grafana Hardware health [AgoricTools by Chainode](https://github.com/Chainode/AgoricTools)
- Stack of monitoring tools, docker configuration [node_tooling by Xiphiar](https://github.com/Xiphiar/node_tooling/)
- Alertmanager telegram bot [alertmanager-bot by metalmatze](https://github.com/metalmatze/alertmanager-bot)
