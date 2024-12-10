<h1 align="center">Apache APISIX Setup</h1>

## What is Apache APISIX?
Apache APISIX is an Open Source API Gateway Management tool. It provides traffic handling for websites, mobile, and IoT applications by offering services like load balancing, dynamic upstream, canary release, fine-grained routing, rate limiting, and more. It is a dynamic, scalable, and high-performance cloud-native API gateway for APIs and microservices.

- [Download Apache APISIX GitHub Repo](https://github.com/apache/apisix.git)

---

## Setup Steps


### Step 1: Installing APISIX
Open your terminal and type the following:
```bash
sudo su
```
Navigate to the `apisix` folder:
```bash
cd apisix
```

### Step 2: Ensure Docker is Installed and Running
Run the following commands to check Docker installation:
```bash
docker -v
docker-compose -v
curl --version
sudo systemctl status docker.service
```

### Step 3: Create Docker Compose File for APISIX Services
Create a `docker-compose.yml` file that launches three APISIX service components: `apisix`, `apisix-dashboard`, and `etcd`. These services interact as follows:

| Service            | Ports                          |
|--------------------|--------------------------------|
| apisix-dashboard   | 9000                           |
| apisix             | 9180, 9080, 9091, 9443, 9092  |
| etcd               | 2379                           |

#### Create Docker Compose File
```bash
touch docker-compose.yml
sudo nano docker-compose.yml
```
Copy and paste the following content:
```yaml
version: "3"

services:
  apisix-dashboard:
    image: apache/apisix-dashboard:3.0.0-alpine
    restart: always
    volumes:
    - ./dashboard_conf/conf.yaml:/usr/local/apisix-dashboard/conf/conf.yaml
    ports:
    - "9000:9000"
    networks:
      apisix:

  apisix:
    image: apache/apisix:${APISIX_IMAGE_TAG:-3.1.0-debian}
    restart: always
    volumes:
      - ./apisix_conf/config.yaml:/usr/local/apisix/conf/config.yaml:ro
    depends_on:
      - etcd
    ##network_mode: host
    ports:
      - "9180:9180/tcp"
      - "9080:9080/tcp"
      - "9091:9091/tcp"
      - "9443:9443/tcp"
      - "9092:9092/tcp"
    networks:
      apisix:

  etcd:
    image: bitnami/etcd:3.4.15
    restart: always
    volumes:
      - etcd_data:/bitnami/etcd
    environment:
      ETCD_ENABLE_V2: "true"
      ALLOW_NONE_AUTHENTICATION: "yes"
      ETCD_ADVERTISE_CLIENT_URLS: "http://etcd:2379"
      ETCD_LISTEN_CLIENT_URLS: "http://0.0.0.0:2379"
    ports:
      - "2379:2379/tcp"
    networks:
      apisix:

networks:
  apisix:
    driver: bridge

volumes:
  etcd_data:
    driver: local
```

#### APISIX Dashboard Configuration
```bash
touch conf.yaml
sudo nano conf.yaml
```
Copy and paste the following content:
```yaml
...
conf:
  listen:
    host: 0.0.0.0     # `manager api` listening ip or host name
    port: 9000          # `manager api` listening port
  allow_list:           # If we don't set any IP list, then any IP access is allowed by default.
    - 0.0.0.0/0
  etcd:
    endpoints:          # supports defining multiple etcd host addresses for an etcd cluster
      - "http://etcd:2379"
                          # yamllint disable rule:comments-indentation
                          # etcd basic auth info
    # username: "root"    # ignore etcd username if not enable etcd auth
    # password: "123456"  # ignore etcd password if not enable etcd auth
    mtls:
      key_file: ""          # Path of your self-signed client side key
      cert_file: ""         # Path of your self-signed client side cert
      ca_file: ""           # Path of your self-signed ca cert, the CA is used to sign callers' certificates
    # prefix: /apisix     # apisix config's prefix in etcd, /apisix by default
  log:
    error_log:
      level: warn       # supports levels, lower to higher: debug, info, warn, error, panic, fatal
      file_path:
        logs/error.log  # supports relative path, absolute path, standard output
                        # such as: logs/error.log, /tmp/logs/error.log, /dev/stdout, /dev/stderr
    access_log:
      file_path:
        logs/access.log  # supports relative path, absolute path, standard output
                         # such as: logs/access.log, /tmp/logs/access.log, /dev/stdout, /dev/stderr
                         # log example: 2020-12-09T16:38:09.039+0800	INFO	filter/logging.go:46	/apisix/admin/routes/r1	{"status": 401, "host": "127.0.0.1:9000", "query": "asdfsafd=adf&a=a", "requestId": "3d50ecb8-758c-46d1-af5b-cd9d1c820156", "latency": 0, "remoteIP": "127.0.0.1", "method": "PUT", "errs": []}
  security:
      # access_control_allow_origin: "http://httpbin.org"
      # access_control_allow_credentials: true          # support using custom cors configration
      # access_control_allow_headers: "Authorization"
      # access_control-allow_methods: "*"
      # x_frame_options: "deny"
      content_security_policy: "default-src 'self'; script-src 'self' 'unsafe-eval' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; frame-src *"  # You can set frame-src to provide content for your grafana panel.

authentication:
  secret:
    secret              # secret for jwt token generation.
                        # NOTE: Highly recommended to modify this value to protect `manager api`.
                        # if it's default value, when `manager api` start, it will generate a random string to replace it.
  expire_time: 3600     # jwt token expire time, in second
  users:                # yamllint enable rule:comments-indentation
    - username: admin   # username and password for login `manager api`
      password: admin
    - username: user
      password: user

plugins:                          # plugin list (sorted in alphabetical order)
  - api-breaker
  - authz-keycloak
  - basic-auth
  - batch-requests
  - consumer-restriction
  - cors
  # - dubbo-proxy
  - echo
  # - error-log-logger
  # - example-plugin
  - fault-injection
  - grpc-transcode
  - hmac-auth
  - http-logger
  - ip-restriction
  - jwt-auth
  - kafka-logger
  - key-auth
  - limit-conn
  - limit-count
  - limit-req
  # - log-rotate
  # - node-status
  - openid-connect
  - prometheus
  - proxy-cache
  - proxy-mirror
  - proxy-rewrite
  - redirect
  - referer-restriction
  - request-id
  - request-validation
  - response-rewrite
  - serverless-post-function
  - serverless-pre-function
  # - skywalking
  - sls-logger
  - syslog
  - tcp-logger
  - udp-logger
  - uri-blocker
  - wolf-rbac
  - zipkin
  - server-info
  - traffic-split
...
```

#### APISIX Configuration
```bash
touch config.yaml
sudo nano config.yaml
```
Copy and paste the following content:
```yaml
...
apisix:
  node_listen: 9080              # APISIX listening port
  enable_ipv6: false

  enable_control: true
  control:
    ip: "0.0.0.0"
    port: 9092

deployment:
  admin:
    allow_admin:               # http://nginx.org/en/docs/http/ngx_http_access_module.html#allow
      - 0.0.0.0/0              # We need to restrict ip access rules for security. 0.0.0.0/0 is for test.

    admin_key:
      - name: "admin"
        key: edd1c9f034335f136f87ad84b625c8f1
        role: admin                 # admin: manage all configuration data

      - name: "viewer"
        key: 4054f7cf07e344346cd3f287985e76a2
        role: viewer

  etcd:
    host:                           # it's possible to define multiple etcd hosts addresses of the same etcd cluster.
      - "http://etcd:2379"          # multiple etcd address
    prefix: "/apisix"               # apisix configurations prefix
    timeout: 30                     # 30 seconds

plugin_attr:
  prometheus:
    export_addr:
      ip: "0.0.0.0"
      port: 9091
...
```

### Step 4: Start the APISIX Services
Start all APISIX-related services using:
```bash
docker-compose up -d
docker-compose ps
```
If services fail to start, stop the container:
```bash
docker-compose down
sudo systemctl stop etcd
```

### Step 5: Ensure APISIX Service is Running
Validate that APISIX is running with the following API request:
```bash
ifconfig
```
Replace `<inet_ip>` with the `inet` IP found in the output:
```bash
curl "http://<inet_ip>:9180/apisix/admin/services/" -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1'
```

### Step 6: Create Upstreams for Services
Define upstreams for your services:

#### Example: Service
```bash
curl http://<inet_ip>:9180/apisix/admin/upstreams -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X POST -d '{
  "nodes": [
    {
      "host": "172.21.0.1",
      "port": 8001,
      "weight": 1
    }
  ],
  "type": "roundrobin",
  "scheme": "http",
  "pass_host": "pass"
}'
```

### Step 7: Create Routes for Services
#### Example: Route
```bash
curl http://<inet_ip>:9180/apisix/admin/routes -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X POST -d '{
  "uri": "/your_path_url/*",
  "upstream_id": "<your_path_url_upstream_id>",
  "status": 1
}'
```

Repeat Steps 6 and 7 for all services:

| Service   | Port  |
|-----------|-------|
| your_service_name  | 8002  |
| your_service_name     | 8003  |
| your_service_name | 8004  |
| your_service_name   | 8005  |

---

```bash
uvicorn app.asgi:app --reload --host  0.0.0.0 --port your_port_here
```

### Step 8: Validate Upstreams and Route using APISIX Dashboard
We can access the APISIX Dashboard using the following details. Once logged in you can verify the Routes and Upstreams that we created earlier as shown in below screenshots.

<img src="Reference Snaps/reference_image1.png">

```bash
http://127.0.0.1:9000/user/login
```

```bash
username - admin
password - admin

```

You can see your created routes here:

<img src="Reference Snaps/reference_image2.png">

You can see your created Upstream here:

<img src="Reference Snaps/reference_image3.png">



Follow these steps to configure and run APISIX with your services!

