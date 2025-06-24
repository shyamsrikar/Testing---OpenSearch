# Testing - OpenSearch

## What is Opensearch?

- OpenSearch is a free and open-source tool used to search, analyze, and visualize data. It helps you find information quickly from large amounts of text or logs.
- It has two main parts:
  1) OpenSearch – the engine that stores and searches data.
  2) OpenSearch Dashboards – a web UI to see and explore that data in graphs and charts.

## Before installing OpenSearch, you need a server to run it. Here’s how to set up an Ubuntu server on AWS:

- 1) Login to AWS Console and go to EC2 Dashboard.
     
- 2) Click "Launch Instance".
     
![Screenshot from 2025-06-24 15-17-32](https://github.com/user-attachments/assets/7ea57812-4fca-4339-a487-20f1da37c552)

- 3) Choose AMI: Select Ubuntu Server 24.04 LTS (HVM), SSD Volume Type.
     
![Screenshot from 2025-06-24 15-17-50](https://github.com/user-attachments/assets/7580e207-04c8-425c-a312-24063d0ec5d3)

- 4) Instance Type: Choose t3.xlarge or a higher type if needed.
     
- 5) Key Pair: Select or create a key pair to access your server.


![Screenshot from 2025-06-24 15-18-11](https://github.com/user-attachments/assets/2f4acf91-45a7-42b9-a5fd-c0c5def8d2de)
     

- 6) Network Settings:
    - i) Allow SSH (port 22)
    - ii) Allow Custom TCP on port 9200  (For Running OpenSearch)
    - iii) Allow Custom TCP on port 5601  (For Dashboards)

![Screenshot from 2025-06-24 15-19-22](https://github.com/user-attachments/assets/b74ad84b-4d78-4e6b-bcac-f8ba6e10da46)

- 7) Choose volume atleast 20GB.
 
![Screenshot from 2025-06-24 15-19-55](https://github.com/user-attachments/assets/97095ee6-2753-4491-b4b4-5f1e38a032c6)

- 8) Launch Instance.
     
- 9) After it’s running, connect to it using:

![Screenshot from 2025-06-24 15-20-38](https://github.com/user-attachments/assets/20d316be-033e-49bf-8247-4f2ba7622195)

## Replace with actual public ip of instance
```
 ssh -i your-key.pem ubuntu@<your-public-ip>
```

After Connecting:

![Screenshot from 2025-06-24 16-08-51](https://github.com/user-attachments/assets/45a8d9f5-91aa-49f9-8697-600f665395b1)


# 🛠️ Installation Steps for OpenSearch on Ubuntu Server After Connecting to the Instance

## To install OpenSearch on Ubuntu, follow these steps:

### ✅ Step 1: Update System

Run a command to update the list of software packages and upgrade them to the latest version.


```
 sudo apt update && sudo apt upgrade -y
```

### ✅ Step 2:  Install Docker & Docker Compose

```
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable docker
sudo systemctl start docker

```


### ✅ Step 3: Create Project Folder

```
mkdir ~/opensearch-docker
cd ~/opensearch-docker
```

### ✅ Step 4: Create docker-compose.yml

```
nano docker-compose.yml

```
- Paste the following fully working content:

```
version: '3'
services:
  opensearch:
    image: opensearchproject/opensearch:2.14.0
    container_name: opensearch
    environment:
      - discovery.type=single-node
      - plugins.security.disabled=true
      - OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m
      - OPENSEARCH_INITIAL_ADMIN_PASSWORD=Shyam@184239!
    ports:
      - "9200:9200"
    volumes:
      - opensearch-data:/usr/share/opensearch/data

  dashboards:
    image: opensearchproject/opensearch-dashboards:2.14.0
    container_name: dashboards
    ports:
      - "5601:5601"
    environment:
      - OPENSEARCH_HOSTS=["http://opensearch:9200"]
      - DISABLE_SECURITY_DASHBOARDS_PLUGIN=true

volumes:
  opensearch-data:

```
- Save and exit (Ctrl+O Enter, then Ctrl+X).
    
### ✅ Step 5: Run Containers

```
sudo docker-compose up -d

```

![Screenshot from 2025-06-24 15-26-07](https://github.com/user-attachments/assets/0fd680df-8d1d-46d9-b441-956c6544fd79)


### ✅ Step 6: Verify Everything is Running

- Containers
  
```
sudo docker-compose ps

```

![Screenshot from 2025-06-24 15-30-05](https://github.com/user-attachments/assets/33eaa57f-251a-477c-9846-4f2b4ef21bc3)


- Test OpenSearch API:

```
curl -u admin:Shyam@184239! http://localhost:9200

```

- Expected output:
  
```
{
"name" : "...",
"cluster_name" : "docker-cluster",
...
}
```



# Optional

### ✅ Step 1: Index Sample Document Testing:

- Create a new index named test-index and add a document:

```
curl -X POST "http://localhost:9200/test-index/_doc/1" \
  -u admin:Shyam@184239! \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Shyam",
    "location": "Hyderabad"
}'
```

- Expected output:

```
{
  "_index": "test-index",
  "_id": "1",
  "result": "created",
  ...
}

```

![Screenshot from 2025-06-24 15-39-31](https://github.com/user-attachments/assets/98d9edb7-7c53-4b85-b884-03f0fdbbc7d0)


### ✅ Step 2: Retrieve the Document by ID

```
curl -X GET "http://localhost:9200/test-index/_doc/1" \
  -u admin:Shyam@184239!

```

- Expected output:

```
{
  "_index": "test-index",
  "_id": "1",
  "_source": {
    "name": "Shyam",
    "location": "Hyderabad"
  }
}

```
![Screenshot from 2025-06-24 15-40-07](https://github.com/user-attachments/assets/5655f6f2-3650-420b-8bef-ab8c7215f4a2)


### ✅ Step 3: 
- Visit: http://<your-ec2-ip>:5601
- Login: admin / Shyam@184239!
- Go to: Dev Tools → Console

![Screenshot from 2025-06-24 15-33-28](https://github.com/user-attachments/assets/616f6768-6d27-4cd5-82b9-099310096891)

- Run queries like:

```
GET test-index/_search
{
  "query": {
    "match": {
      "name": "Shyam"
    }
  }
}
```
- Expected Output:

 ![Screenshot from 2025-06-24 16-26-51](https://github.com/user-attachments/assets/70330502-9f61-4539-afb3-7df7d0ebc523)
 
