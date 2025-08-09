---
title: "Prometheus and Grafana Monitoring web server with Docker on AWS EC2"
datePublished: Sat Aug 09 2025 05:10:07 GMT+0000 (Coordinated Universal Time)
cuid: cme3sq2rv000o02jsenxq6a4b
slug: prometheus-and-grafana-monitoring-web-server-with-docker-on-aws-ec2
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1754716118890/30838711-a95f-4494-a17d-018874b195c2.png
tags: web-app, monitoring, prometheus, grafana

---

This lab shows how to create monitoring with Prometheus+Grafana for better monitoring for webapp server on EC2 using docker.

This is the diagram for this lab

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754715745367/233df145-0b38-41d3-bb3e-494bd2cdbf13.png align="center")

## 1\. Create web-app EC2 Server

Create EC2 for web-app server on EC2 using new or created KeyPair. I used ubuntu image with t2.micro in this lab for minimum cost.

![](https://file.notion.so/f/f/52a70352-2e19-4432-b891-a97df8bdac28/17c76778-e1a0-47cc-a9cd-a97a4b8b9851/image.png?table=block&id=249dd0b3-f858-80e5-b995-e3e5bc979473&spaceId=52a70352-2e19-4432-b891-a97df8bdac28&expirationTimestamp=1754740800000&signature=gp6gn5efnpdypW-qWJVlD5fqlmTE0kWrWnpbYtbV6w0&downloadName=image.png align="left")

Create new security group allowing SSH, HTTP and Port 5000

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754713247421/64466be5-61b9-4dd6-835f-cd99fcf4b832.png align="center")

In here for port 5000 if you use Elastic IP for Webapp server you can use Source type as that Prometheus Server EIP. If you don’t use Elastic IP for webapp server the public IP will change every time the node restarts.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754713297122/ca5c3d05-6b0b-4d5c-b868-36f5e0f2a872.png align="center")

Under Advanced Details, use the user data, so then required packages are installed when server is up. Then Launch Instance

```bash
#!/bin/bash

# Update and install Docker + Docker Compose
apt update -y
apt install -y docker.io docker-compose

# Add the 'ubuntu' user to the docker group
usermod -aG docker ubuntu

# Enable and start Docker
systemctl enable docker
systemctl start docker
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754713409810/73c63aad-e8d6-4dc8-a9f7-b599e24a7a4d.png align="center")

After logging in to the web-app server using the public IP and keypair. Create webapp directory and go inside that directory

```bash
mkdir webapp && cd webapp
```

Create [`app.py`](http://app.py) file for webapp test using below code.

```bash
from flask import Flask, Response
app = Flask(__name__)

@app.route("/")
def index():
    return "Web server is running"

@app.route("/metrics")
def metrics():
    return Response('webapp_up 1\n', mimetype='text/plain')

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

Create `Dockerfile` from below. You can take reference from [https://docs.docker.com/reference/dockerfile/](https://docs.docker.com/reference/dockerfile/) for `Dockerfile`.

```bash
FROM python:3.9-slim
WORKDIR /app
COPY app.py .
RUN pip install flask
CMD ["python", "app.py"]
```

Create `docker-compose.yml`. You can take reference from [https://docs.docker.com/reference/compose-file/](https://docs.docker.com/reference/compose-file/) for `docker-compose.yml`

```bash
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
```

Run

```bash
docker-compose up -d --build
```

`-d` means detach mode and `--build` means build the image new instead of reusing the old image. After that use `docker ps` to check the running docker.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754713710614/933560c9-ba65-447b-9e73-6056b92c8d78.png align="center")

## 2\. Create Prometheus server On EC2 with docker

Create another EC2 for Prometheus server using the above steps. But this time instead of opening port 5000 open port 9090 for Prometheus.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754713733412/6174668a-533f-4aaf-a338-a7c71f111c7f.png align="center")

After connecting to Prometheus server,

```bash
mkdir prometheus && cd prometheus
```

Create `prometheus.yml` file for config of target server. You can take reference from here [https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape\_config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape_config)

```bash
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'web-app'
    static_configs:
      - targets: ['<WEB_EC2_PUBLIC_IP>:5000']
```

Create `docker-compose.yml`

```bash
version: '3'
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
```

After that check http://&lt;PROMETHEUS\_PUBLIC\_IP&gt;:9090/targets

you should see the webserver as targe

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754713915774/c6850dcd-7351-46f6-bb32-531473f1341d.png align="center")

## 3\. Create Grafana server On EC2 with docker

Create EC2 as above steps for Grafana server. But this time open port 3000 for Grafana Serve

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754713931859/eaafb140-3206-47ca-af8b-eec7d14d92e0.png align="center")

Login to Grafana Server and create grafana directory

```bash
mkdir grafana && cd grafana
```

Create `docker-compose.yml` . I want to avoid hardcoding the grafana password in docker-compose file so I will use another method by creating `.env` and add the line `GRAFANA_ADMIN_PASSWORD=admin123` use your desired password. Here I used admin123 for grafana admin password. Use `.gitignore` to ignore uploading to git for `.env` file so then your password is not uploaded to git.

```bash
version: '3'
services:
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_ADMIN_PASSWORD}
```

Then run the following

```bash
docker-compose up -d
```

You can check http://&lt;GRAFANA\_PUBLIC\_IP&gt;:3000

if Grafana is up, use  
username: admin  
password: &lt;your input password&gt;

Go to `Connections>Data Sources>Prometheus` to show data send from Prometheus in Grafana

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754714134000/694066d6-56e1-4698-8979-470377cf8715.png align="center")

Go to Dashboard&gt;New Dashboard&gt;Add Visualization&gt;(Select Data Source as Prometheus). You can use metrics for example up for server up in visualization. Then save dashboard

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754714146033/1fe58e10-6e8e-4212-a71d-e86e89747649.png align="center")

You can see as below as server uptime.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754714156718/086f05af-2cae-4909-a5c2-b0851847dcd9.png align="center")

You can also integrate alerting with PagerDuty, Slack etc. I use PagerDuty to test alert

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754714168687/22363602-6012-41d7-be69-fa481081e3ca.png align="center")

Go to Alert&gt;New Alert Rule. Use metrics as up or webapp\_up and when query is Equal to 0 the alert will trigger. Create the required Alert Folder, Evaluation Grp and most importantly create contact points.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754714184656/751102c1-80c7-40df-834b-efc052c83fda.png align="center")

Go to Create contact points and choose integration as PagerDuty. And you have to copy the Integration key from PagerDuty and put it here.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754714192758/466d7944-fb1c-4a68-a4fa-d28895c310cf.png align="center")

In Pager Duty Go to Services&gt;New Service and create a service with desired name.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754714214013/1d36a379-eaf2-4fa7-81a2-33c5bbe72a13.png align="center")

Choose the Service created and go to Integrations and Add integration. You can choose Events API V2 or Prometheus. Any of them working. Copy the Integration key from Pager Duty and paste in Grafana Contact Point.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754714353930/8d40aec3-b9d8-4d59-8813-1daa3e07fa97.png align="center")

Try stopping the webapp container using `docker stop <container ID>`. After some time you will receive an email from Pager Duty as below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754714387900/25ef3f3d-6f2e-4f4b-bc0f-962bad437ffd.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754714394477/5efc3459-3964-4956-b544-3e1acf36e753.png align="center")

After all the testing has been finished don’t forget to delete all three EC2 that you used for this lab to avoid additional cost.