# Nexus Repository Notes
## Installation:
### Requiremnts:
![R2](./Requirement.png)
### Linux OS:
*** Initial Setup: ***
**Instal JDK:**


- **Step 1 — Installing Java**

Newer versions of Nexus have updated their Java requirements.

    Supported Java Versions: “Nexus Repository is tested on and supports OpenJDK and requires Java 21. Nexus Repository is compatible with both Intel and AMD CPU architectures. As of release 2.78.0 the Nexus Repository bundle includes the recommended JVM. See Java Compatibility Matrix.”

While the Nexus bundle includes a JVM, installing OpenJDK 21 on your system ensures that all environment variables are correctly set and provides a fallback if you choose to run Nexus with an external JDK.

Update your package index and install OpenJDK 21:
```bash
$ sudo apt update
$ sudo apt install openjdk-21-jdk -y
```
```bash
$ java -version
openjdk version "21.0.9" 2025-10-21
OpenJDK Runtime Environment (build 21.0.9+10-Ubuntu-124.04)
OpenJDK 64-Bit Server VM (build 21.0.9+10-Ubuntu-124.04, mixed mode, sharing)
```
Output should indicate OpenJDK 21.

- **Step 2 — Downloading Nexus**

Navigate to the /opt directory:
```bash
$ cd /opt
```
Download the specific Nexus release (version 3.87.1–01) using curl. The -O flag saves the file with its original name, and -L ensures redirects are followed if necessary:
```bash
$ sudo curl -L -O https://download.sonatype.com/nexus/3/nexus-3.87.1-01-linux-x86_64.tar.gz
```
Make sure it is downloaded
```bash
$ ll
total 457364
drwxr-xr-x  3 root root      4096 Jan  2 15:11 ./
drwxr-xr-x 22 root root      4096 Dec 27 22:19 ../
drwxr-xr-x  4 root root      4096 Dec 27 22:20 digitalocean/
-rw-r--r--  1 root root 468321562 Jan  2 15:25 nexus-3.87.1-01-linux-x86_64.tar.gz
```
Extract the archive:
```bash
$ sudo tar -xvzf nexus-3.87.1-01-linux-x86_64.tar.gz
```
Make sure it is extracted (Two directories are extracted: nexus-3.87.1–01 and sonatype-work)
```bash
$ ll
total 457372
drwxr-xr-x  5 root root      4096 Jan  2 15:26 ./
drwxr-xr-x 22 root root      4096 Dec 27 22:19 ../
drwxr-xr-x  4 root root      4096 Dec 27 22:20 digitalocean/
drwxr-xr-x  6 root root      4096 Jan  2 15:26 nexus-3.87.1-01/
-rw-r--r--  1 root root 468321562 Jan  2 15:25 nexus-3.87.1-01-linux-x86_64.tar.gz
drwxr-xr-x  3 root root      4096 Dec  3 21:13 sonatype-work/
```
Rename the extracted directory to nexus for easier management:
```bash
$ sudo mv nexus-3.87.1-01 nexus
```
Clean up the tarball to save space:
```bash
$ sudo rm nexus-3.87.1-01-linux-x86_64.tar.gz
```
- **Step 3 — Creating a Dedicated User**

Create a new user named nexus. You will be prompted to set a password and fill in user details (you can press ENTER to skip the details):
```bash
$ sudo adduser nexus
```
Next, add the new nexus user to the sudo group to grant it root privileges:
```bash
$ sudo usermod -aG sudo nexus
```
Finally, change the ownership of the Nexus installation and data directories to this new user:
```bash
$ sudo chown -R nexus:nexus /opt/nexus
$ sudo chown -R nexus:nexus /opt/sonatype-work
```
- **Step 4 — Configuring Nexus as a Service**

Create a systemd unit file to manage the Nexus process.
```bash
$ sudo vi /etc/systemd/system/nexus.service
```
Paste the following configuration:
```ini
[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort
[Install]
WantedBy=multi-user.target
```
Save and exit the file.
Get Mohamed Salah’s stories in your inbox

Join Medium for free to get updates from this writer.

Next, explicitly set the run user in the Nexus run configuration:
```bash
$ vi /opt/nexus/bin/nexus
```
Uncomment the run_as_user line and set it to nexus:
```ini
run_as_user="nexus"
```
- **Step 5 — Starting the Service**

Reload the systemd daemon to recognize the new service:
```bash
$ sudo systemctl daemon-reload
```
Start and enable Nexus:
```bash
$ sudo systemctl start nexus
$ sudo systemctl enable nexus
```
- **Step 6 — Accessing Nexus**

Nexus takes a few minutes to bootstrap. You can watch the logs to see when it is ready:
```bash
$ tail -f /opt/sonatype-work/nexus3/log/nexus.log
```
- **Step 7 - Browse web**
```text
http://192.168.56.186:8081
user: admin
Password: cat /opt/sonatype-work/nexus3/admin.password
```
### Docker Container:

**1. Download the Docker image using following commands..**
```bash
docker pull sonatype/nexus
``` 
**2. Create directories for persistent data**

Nexus stores all repositories and configuration under /nexus-data.

Create a safe local folder and adjust permissions:
```bash
sudo mkdir -p /opt/nexus-data
sudo chown -R 200:200 /opt/nexus-data
```


**3. Run Nexus3 container**

You can launch Nexus directly using Docker:
```bash
sudo docker run -d \
  --name nexus3 \
  -p 8081:8081 \
  -v /opt/nexus-data:/nexus-data \
  sonatype/nexus3:latest
```

**4. To determine the port that the container is listening on:**
```bash
docker ps nexus
```
 
**5. To test:**
```bash
curl http://localhost:8081/service/local/status
```
 
> Notes:
> 1. Default credentials are: ***admin / admin123***
> 
>  
> 2. It can take some time (2-3 minutes) for the service to launch in a new container. You can tail the log to determine once Nexus is ready:
> ```bash
> docker logs -f nexus
> ```
> 3. Four environment variables can be used to control the JVM arguments
> CONTEXT_PATH, passed as -Dnexus-webapp-context-path. 
> This is used to define the
> URL which Nexus is accessed.
> MAX_HEAP, passed as -Xmx. Defaults to 768m.
> MIN_HEAP, passed as -Xms. Defaults to 256m.
> JAVA_OPTS. Additional options can be passed to the JVM via this variable.
> Default: -server -XX:MaxPermSize=192m -Djava.net.preferIPv4Stack=true.
> LAUNCHER_CONF. A list of configuration files supplied to the
> Nexus bootstrap launcher. Default: ./conf/jetty.xml ./conf/jetty-requestlog.xml


## Add SSL Key:
**Use Reverse Engine method (nginx):**
Run Behind a Reverse Proxy

Using a reverse proxy in front of Nexus Repository is a highly recommended and common practice for several compelling reasons, covering security, performance, flexibility, and operational efficiency. This section provides guidance on how to configure a reverse proxy servers to work with Nexus Repository.

Here are some of the key benefits:

 - **SSL/TLS Termination (HTTPS Access)**
 - **Enhanced Security**
 - **Standardization**
 - **Load Balancing and High Availability**
```bash
apt update
apt install nginx
cd /etc/nginx
mkdir certs
cd /etc/nginx/sites-available/
vim nexus.conf
```
Copy standard SSL config 


```bash
server {
    listen 80;
    server_name repo.samen.com;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name repo.samen.com 192.168.56.186;

    ssl_certificate     /etc/nginx/certs/nexus.crt;
    ssl_certificate_key /etc/nginx/certs/nexus.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    client_max_body_size 2G;

    # Nexus UI/API fallback
    location / {
        proxy_pass http://127.0.0.1:8081/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_redirect http://localhost:8081/ $scheme://$host/;
        proxy_http_version 1.1;
        proxy_send_timeout 150;
        proxy_read_timeout 320;
    }
}

```

so
```bash
systemctl daemon-reload
systemctl restart nginx
```
***Example for self sign SSL with url and use docker rep 5002 port:***
- Step 1: Create a new self-signed cert with SAN

Create an OpenSSL config file, for example san.cnf:
```ini
cat > san.cnf <<EOF
[ req ]
default_bits       = 2048
prompt             = no
default_md         = sha256
distinguished_name = dn
x509_extensions    = v3_req

[ dn ]
CN = repo.samen.com

[ v3_req ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = repo.samen.com
IP.1  = 192.168.56.186
EOF
```
- Step 2: Generate the certificate
```bash
openssl req -x509 -nodes -days 3650 \
  -newkey rsa:2048 \
  -keyout /etc/nginx/certs/nexus.key \
  -out /etc/nginx/certs/nexus.crt \
  -config san.cnf
```

This generates a key + cert valid for 10 years

Includes repo.samen.com (and your server IP) in the SAN

- Step 3: Configure Nginx
```bash
ssl_certificate     /etc/nginx/certs/nexus.crt;
ssl_certificate_key /etc/nginx/certs/nexus.key;
```

Reload Nginx:
```bash
sudo nginx -t
sudo systemctl reload nginx
```
- Step 4: Trust this cert on Docker hosts

On every VM that will pull/push images:
```bash
sudo mkdir -p /etc/docker/certs.d/repo.samen.com
sudo cp /etc/nginx/certs/nexus.crt /etc/docker/certs.d/repo.samen.com/ca.crt
sudo systemctl restart docker
```
- Step 5: Test
```bash
docker login repo.samen.com
docker pull repo.samen.com/nginx:latest
```

Now Docker will accept the certificate because the SAN matches repo.samen.com.


## BLOB STORE
### 1. What is a Blob in Nexus?

In **Nexus Repository**, a **blob** is essentially the storage unit that holds the raw content of artifacts (like JARs, npm packages, Docker images, etc.).

- Think of it as a **file on disk** managed by Nexus.  
- Blobs are stored in a **blob store**, which can be either **file-based** or **S3 (or other supported cloud storage)**.  
- Every artifact version is stored as a blob in the blob store, and Nexus keeps a reference to it in its database.

> **Note:** A blob is **not the repository itself**, but the **underlying storage unit** for your artifacts.

---

### 2. Types of Blob Stores in Nexus

There are mainly **two types** of blob stores:

1. **File Blob Store (default)**
   - Stores blobs on the local filesystem of the Nexus server.
   - Example path: `/nexus-data/blobs/default/`.

2. **S3 Blob Store (or other cloud storage)**
   - Stores blobs in cloud object storage (S3, Google Cloud Storage, Azure Blob Storage).  
   - Useful for large artifacts and HA setups.

3. **Hybrid/Other types** depending on Nexus version (less common).

---

### 3. How to Create a Blob Store in Nexus

You **don’t create blobs manually**, you create **blob stores**, and then Nexus stores blobs there automatically when you upload artifacts.  

### Step 1: Log in to Nexus UI
- Go to `http://<nexus-host>:8081/`
- Log in as an admin user. 

### Step 2: Go to Blob Stores
- Navigate to: **Administration → Repository → Blob Stores**

### Step 3: Create a New Blob Store
1. Click **Create blob store**.
2. Fill the form:
   - **Name:** e.g., `maven-blobs`
   - **Type:** `File` or `S3`
   - **Path (for File type):** `/nexus-data/blobs/maven`  
   - For S3: Provide bucket name, region, credentials, etc.
3. Click **Create**.

### Step 4: Assign Blob Store to Repository
- When creating a new repository (e.g., Maven, npm, Docker), select the blob store where its artifacts will be stored.

---

### 4. How Blobs Are Managed

- **Automatic:** When you upload an artifact, Nexus creates a blob in the assigned blob store.  
- **Checksum-based:** Nexus calculates a SHA1/MD5 hash and stores a blob with a unique name.  
- **Repository-agnostic:** The same blob can be shared between multiple repositories if content is identical.

> **Tip:** Always assign repositories to a dedicated blob store to organize storage and simplify backups.

## Create Repository:

### Docker Repo:

#### Step 1: Enable Docker (if not already)
In Nexus 3, Docker support is enabled by default. No extra plugin is needed.

---

#### Step 2: Create a Docker (hosted) repository
This is where your images will be stored.

1. Go to **Settings** (gear icon)  
2. Click **Repositories**  
3. Click **Create repository**  
4. Choose **docker (hosted)**  

##### Repository configuration
- **Name:** `docker-hosted` (example)  
- **HTTP:** Choose a port, e.g., `7001`  
  *(Make sure this port is open on the server firewall)*  
- **Enable Docker V1 API:** Leave unchecked *(only needed for very old Docker clients)*  
- **Blob store:** `default` (or your custom one)  
- **Deployment policy:** `Allow redeploy` (recommended)  

Click **Create repository**

---

#### Step 3: (Optional but recommended) Create a Docker proxy repository
This lets Nexus cache Docker Hub images.

1. Click **Create repository**  
2. Choose **docker (proxy)**  

##### Settings
- **Name:** `docker-proxy`  
- **Remote storage:** `https://registry-1.docker.io`  
- **HTTP port:** `7002`  

Click **Create repository**

---

#### Step 4: (Optional) Create a Docker group repository
This is best practice so users use a single endpoint.

1. Click **Create repository**  
2. Choose **docker (group)**  

##### Settings
- **Name:** `docker-group`  
- **HTTP port:** `7000`  
- **Member repositories:**  
  - `docker-hosted`  
  - `docker-proxy`  

Click **Create repository**

---
  `***VERY VERY IMPORTANT NOTE***`
**`Best Practice for config docker repo:`**
- 1. set port for nginx to mapping with internal nexus ports ``external nginx port 5001:7001 internal nexus port``
- 2. create 2 blob one for hosted and other one to group and proxy
- 3. pull just with repo.samen.com:5001 and pull just from repo.samen.com:5000

```nginx
server {
    listen 80;
    server_name repo.samen.com;
    return 301 https://repo.samen.com$request_uri;
}

server {
    listen 443 ssl;
    server_name repo.samen.com;

    ssl_certificate     /etc/nginx/certs/nexus.crt;
    ssl_certificate_key /etc/nginx/certs/nexus.key;

    client_max_body_size 5G;

    location / {
        proxy_pass http://127.0.0.1:8081/;

        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Port 443;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Connection "";

        proxy_buffering off;
        proxy_request_buffering off;

        proxy_read_timeout 300;
    }
}



server {
    listen 5001 ssl;
    server_name repo.samen.com;

    ssl_certificate     /etc/nginx/certs/nexus.crt;
    ssl_certificate_key /etc/nginx/certs/nexus.key;

    client_max_body_size 0;

    proxy_http_version 1.1;
    proxy_buffering off;
    proxy_request_buffering off;

    location /v2/ {
        proxy_pass http://127.0.0.1:7001;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

server {
    listen 5000 ssl;
    server_name repo.samen.com;

    ssl_certificate     /etc/nginx/certs/nexus.crt;
    ssl_certificate_key /etc/nginx/certs/nexus.key;

    client_max_body_size 0;

    proxy_http_version 1.1;
    proxy_buffering off;
    proxy_request_buffering off;

    location /v2/ {
        proxy_pass http://127.0.0.1:7000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

***[note]: `in OSS version docker-group repository Read-Only and we are not able to push in it and we use docker-hosted in config for access to push and pull`*** 

and:
```bash
sudo mkdir -p /etc/docker/certs.d/repo.samen.com
sudo mkdir -p /etc/docker/certs.d/repo.samen.com:5000 
sudo mkdir -p /etc/docker/certs.d/repo.samen.com:5001
sudo cp /etc/nginx/certs/nexus.crt /etc/docker/certs.d/repo.samen.com/ca.crt
sudo cp /etc/nginx/certs/nexus.crt /etc/docker/certs.d/repo.samen.com:5000/ca.crt
sudo cp /etc/nginx/certs/nexus.crt /etc/docker/certs.d/repo.samen.com:5001/ca.crt
systemctl restart docker

# PULL

docker login repo.samen.com:5000
docker pull repo.samen.com:5000/alpine

#PUSH

docker pull alpine/test   # from docker hub
docker tag alpine repo.samen.com:5001/test/alpine
docker login repo.samen.com:5001
docker push repo.samen.com:5001/test/alpine
```

### **APT Repo:**

This guide explains how to:

- Generate a GPG key on the Nexus server
- Configure an APT hosted repository
- Verify signing
- Configure client access

---

- #### STEP 1 — Create GPG Key on Nexus Server

Login to the Nexus server terminal.

**Install GPG (if not installed)**

```bash
$ sudo apt update
$ sudo apt install gnupg -y
```
Generate New GPG Key
```bash
$ gpg --full-generate-key
```
Choose:

```bash
1) RSA and RSA
```
Key size:
```bash
4096
```
Expiration:
```bash
0
```
Confirm:
```bash
y
```
Enter:

```yml
Real name: Nexus APT

Email: repo@nexus.local

Comment: (leave empty)
```
Set a strong passphrase

The key is now created.

- #### STEP 2 — Get Your Key ID
List secret keys:
```bash
$ gpg --list-secret-keys --keyid-format LONG
```

Example output:

```cs
sec   rsa4096/ABCDEF1234567890 2026-02-11 [SC]
```
Copy the KEYID:

```bash
ABCDEF1234567890
```
- #### STEP 3 — Export PRIVATE Key (Required by Nexus)

Export the private key in armored format:
```bash
$ gpg --export-secret-key --armor ABCDEF1234567890 > nexus-private.key
```
Check file:

```bash
$ cat nexus-private.key
```
You must see:
```bash
-----BEGIN PGP PRIVATE KEY BLOCK-----
...
-----END PGP PRIVATE KEY BLOCK-----
```
Copy the entire content including BEGIN and END lines.

- #### STEP 4 — Export Public Key (For Clients)

```bash
$ gpg --export --armor ABCDEF1234567890 > nexus-public.key
```

Keep this file safe. It will be used on client machines.

- #### STEP 5 — Configure Nexus APT Hosted Repository
Open Nexus UI:
```ruby
http://your-server:8081
```

Go to:

```text
Administration → Repositories → Create repository → apt (hosted)
```
Fill the fields:

A unique identifier for this repository
Example:
```text
apt-hosted
(No spaces)
```
Distribution
Example:
```text
jammy
```
Must match client Ubuntu version.

Examples:

- focal (20.04)

- jammy (22.04)

- noble (24.04)

PGP signing key pair (armored private key)
Paste the FULL content of:

```text
nexus-private.key
```

Including:
```text
-----BEGIN PGP PRIVATE KEY BLOCK-----
...
-----END PGP PRIVATE KEY BLOCK-----
```
`**Important**` Deployment Policy:
```text
Allow redeploy
```

Click `Save`.

- #### STEP 6 — set permissions for repository:



1. Check Admin Roles

Go to:
```pgsql
Security → Users → admin
```

Check assigned roles.

The built-in admin must have:

```pgsql
nx-admin
```
2. Create a Custom Role

Go to:
```pgsql
Security → Roles → Create role
```

Example:
```pgsql
Role ID: apt-hosted-admin
Name: APT Hosted Admin
```

Add these privileges:
```pgsql
nx-repository-view-apt-apt-hosted-*
```

(That is enough for upload + delete + read)

`If not work add all apt-hosted permisions`

If you also want config access:
```pgsql
nx-repository-admin-apt-apt-hosted-*
```

`Save`.

If it has nx-admin, you DO NOT need to add anything else.
- #### STEP 7 — Upload Test Package
1. Open Nexus in browser:
```ruby
   http://192.168.56.186:8081
```

1. Go to:
   Administration (gear icon) → System → API

   Direct URL:
```ruby
   http://192.168.56.186:8081/#admin/system/api
```
`1. In the API page (Swagger UI):`

   `- Expand: Components API`

   `- Find: POST /service/rest/v1/components`

   `- Click: "Try it out"`


1. For APT hosted upload, the endpoint is:
```pgsql
   POST /service/rest/v1/components?repository=apt-hosted
```
1. Full URL becomes:
```ruby
   https://repo.samen.com/service/rest/v1/components?repository=apt-hosted
```
Example curl:
```bash
$ curl -u admin:admin \
  -X POST "http://192.168.56.186:8081/service/rest/v1/components?repository=apt-hosted" \
  -H "accept: application/json" \
  -F "apt.asset=@/path/to/package.deb"
```

### How Other VMs connect to apt repo:

Now we configure client cleanly.

1. Install SSL Certificate (ONLY if using self-signed nginx)

If repo uses self-signed certificate:
```bash
$ echo | openssl s_client -connect repo.samen.com:443 -servername repo.samen.com 2>/dev/null | openssl x509 > repo.samen.com.crt
$ sudo cp repo.samen.com.crt /usr/local/share/ca-certificates/
$ sudo update-ca-certificates
```

If using valid certificate (Let's Encrypt) → skip this step.

2. Install Nexus GPG Public Key

`Copy nexus-public.key to client.`

Then:
```bash
$ sudo gpg --dearmor nexus-public.key
$ sudo mv nexus-public.key.gpg /usr/share/keyrings/nexus-apt.gpg
```

Verify:
```bash
$ gpg --show-keys /usr/share/keyrings/nexus-apt.gpg
```
3. Add Repository

Create file:
```bash
$ sudo nano /etc/apt/sources.list.d/nexus.list
```

Add exactly this:
```conf
deb [signed-by=/usr/share/keyrings/nexus-apt.gpg] https://repo.samen.com/repository/apt-hosted noble main
```

`Save.`

4. Update
```bash
$ sudo apt update
```

You must see:
```ruby
Hit: https://repo.samen.com/repository/apt-hosted noble InRelease
```

No errors.

5. Install Package

Example:
```bash
$ sudo apt install htop
```

``IMPORTANT``
If it installs → everything is correct.

✅ Minimal Lab Version (Skip GPG)

If you don't care about security (lab only):

In /etc/apt/sources.list.d/nexus.list:
```ruby
deb [trusted=yes] http://repo.samen.com/repository/apt-hosted noble main
```

Then:
```bash
$ sudo apt update
```

`This skips GPG completely.`
## How migrate data from nexus to anotherone:

Your Nexus Data Directory Is:
```text
/opt/sonatype-work/nexus3
```
Verify It

Run:
```bash
$ ls /opt/sonatype-work/nexus3


You should see something like:

blobs
cache
db
etc
log
tmp
```

If yes → this is the directory you must migrate.

🚀 For Migration

You must copy:

```text
/opt/sonatype-work/nexus3
```


📦 Example Migration Command

After stopping Nexus:

```bash
$ rsync -avz --progress /opt/sonatype-work/nexus3/ user@new-server:/opt/sonatype-work/nexus3/
```

Then fix ownership:
```bash
$ chown -R nexus:nexus /opt/sonatype-work/nexus3
```

If you want, show me:
```bash
$ ls /opt/sonatype-work/nexus3
```

and I will confirm everything is correct before you migrate.

## BEST PRACTICE BACKUP (Recommended)

🔹 Option 1 — Full Offline Backup (Safest)

This guarantees consistency.

1️⃣ Stop Nexus
```bash
$ systemctl stop nexus
```

or
```bash
$ /opt/nexus/bin/nexus stop
```
2️⃣ Backup the Entire Data Directory
```bash
$ tar -czvf nexus-backup-$(date +%F).tar.gz -C /opt/sonatype-work nexus3
```

OR using rsync:
```bash
$ rsync -avz /opt/sonatype-work/nexus3 /backup/
```
3️⃣ Start Nexus
```bash
$ systemctl start nexus
```


