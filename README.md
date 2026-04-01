
#  CST8915 - Lab 8 - Algonquin Pet Store  
**Student: Sara Mirzaei**

---

## Demo Video  
 **YouTube Link:** [Lab 8 demo](https://youtu.be/ySm3qUNjix8)

This demo video includes:

- Deploying the Algonquin Pet Store application using the updated [aps-all-in-one-Task1.yaml](./aps-all-in-one.yaml)
- Demonstrating MongoDB persistence + replica set
- Demonstrating RabbitMQ persistence

---

# Task 1: Build, Push, and Deploy to Docker Images
All provided Docker images were successfully built from their respective Dockerfiles and pushed to Docker Hub under the sara21167 repository. These images were then used in the Kubernetes deployment configuration. 

----

# Task 2 – Improving and Extending the Deployment

This section explains the enhancements made to improve reliability, persistence, and high availability for MongoDB and RabbitMQ.

---

##  1. MongoDB – High Availability + Persistent Storage

###  Before  
- MongoDB ran with 1 replica 
- No persistent storage → data lost on restart  
- No replica set → no high availability  

###  After Improvements  
#### a. Replica count increased to 3
Enables automatic failover and high availability.

#### b. Added `volumeClaimTemplates`
Each MongoDB pod now receives its own PersistentVolumeClaim, ensuring durable storage.

#### c. Converted MongoDB service to a headless service 
This provides stable DNS names required for replica sets:

```
mongodb-0.mongodb
mongodb-1.mongodb
mongodb-2.mongodb
```

#### d. Initialized the MongoDB Replica Set
Inside `mongodb-0`:

```js
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongodb-0.mongodb:27017" },
    { _id: 1, host: "mongodb-1.mongodb:27017" },
    { _id: 2, host: "mongodb-2.mongodb:27017" }
  ]
})
```

#### **e. Updated Makeline Service connection string**  
Replica-set-aware URI:

```
mongodb://mongodb-0.mongodb:27017,mongodb-1.mongodb:27017,mongodb-2.mongodb:27017/?replicaSet=rs0
```

###  Result  
- MongoDB now survives pod restarts  
- Data is persistent  
- Automatic failover works  
- Fully functional replica set  

---

##  2. RabbitMQ – Persistent Message Storage

###  Before  
- RabbitMQ stored data inside the container filesystem  
- Messages disappeared when the pod restarted  

###  After Improvements  
#### **a. Added `volumeClaimTemplates`**  
Each RabbitMQ pod now gets its own PVC.

#### **b. Mounted the PVC at `/var/lib/rabbitmq`**  
This is where RabbitMQ stores durable queues and messages.

#### **c. Preserved existing ConfigMap mount**  
Plugin configuration still loads correctly.

###  Result  
- RabbitMQ messages survive pod restarts  
- Durable queues behave correctly  
- No data loss  

---

##  3. Azure Managed Service Alternatives

### **a. Azure Cosmos DB (MongoDB API)**  
**Purpose:** Fully managed NoSQL database compatible with MongoDB drivers.  
**Why it’s a good fit:**  
- Automatic backups  
- Multi-region replication  
- 99.999% availability  
- No need to manage StatefulSets or PVCs  

### **b. Azure Service Bus**  
**Purpose:** Fully managed enterprise message broker.  
**Why it’s a good fit:**  
- Durable queues and topics  
- Auto-scaling  
- Dead-letter queues  
- Zero maintenance  
- Replaces RabbitMQ entirely  
