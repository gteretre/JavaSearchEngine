# Load Balancing with NGINX in a Distributed System

## Overview

This document discusses the use of NGINX for load balancing in a distributed system. In the project, we implemented a distributed computational system where multiple instances of a JavaQuery module are deployed across computers in the same network. Each instance provides a RESTful API developed using Java Spark. NGINX acts as the load balancer, distributing incoming requests across these instances.

## NGINX Configuration and Default Algorithm

### Configuration

The NGINX load balancer was configured to distribute requests among multiple backend servers. Below is the `nginx.conf` file used in the project:

```nginx
events {}

http {
    upstream javaquery_backend {
        server 192.168.1.44:8081;  # First container
        server 192.168.1.194:8082; # Second container
    }

    server {
        listen 80;
        location / {
            proxy_pass http://javaquery_backend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
    }
}
```

Additionally, the `Dockerfile` for deploying the NGINX container is as follows:

```dockerfile
# Use the official NGINX image
FROM nginx:latest

# Copy the configuration file into the container
COPY nginx.conf /etc/nginx/nginx.conf

# Expose port 80 for incoming requests
EXPOSE 80
```

### Default Load Balancing Algorithm

By default, NGINX uses a round-robin algorithm for load balancing. This method cyclically distributes incoming requests to the available backend servers. This approach ensures an even distribution of requests, preventing a single server from becoming a bottleneck.

#### Advantages of the Round-Robin Algorithm

- **Simplicity**: The round-robin algorithm is easy to implement and requires no additional configuration.
- **Fairness**: It distributes requests evenly across all servers, ensuring balanced resource utilization.
- **Scalability**: New servers can be added easily to the backend pool.

## Data Structures in the Project

The JavaQuery module processes and compares data stored in three different formats:

1. **JSON Files**: Data stored in JSON format, representing hierarchical structures.
2. **Dictionary Files**: Files organized as a dictionary with key-value mappings.
3. **Folder-Based Files**: Nested folder structures containing files for specific subsets of data.

Each module instance has a local copy of these data structures. Queries are processed independently by each module using these local copies.

## Alternative Load Balancing Algorithms

While the current implementation uses the default round-robin algorithm, NGINX supports several other load balancing methods:

- **Least Connections**: Directs traffic to the server with the fewest active connections, which can be beneficial for requests with varying processing times.
- **IP Hash**: Determines which server receives the request based on the client's IP address, ensuring session persistence.
- **Weighted Round-Robin**: Allows assigning different weights to servers based on their capacity.

These algorithms can be implemented by adding specific directives to the `upstream` block in the NGINX configuration.

## Testing with Hazelcast and IMap

To evaluate an alternative storage solution, Hazelcast's distributed data grid was implemented. Using Hazelcast's IMap, a cluster was set up where data was distributed and indexed across nodes. The JavaQuery modules connected to the cluster as clients, fetching data from this distributed source. This architecture offers several benefits:

- **Distributed Storage**: Eliminates the need for maintaining complete local copies of data on each server.
- **Scalability**: Hazelcast automatically distributes data across multiple machines, supporting larger datasets.
- **Flexibility**: Clients can dynamically join or leave the cluster.
- **Fault Tolerance**: Data is replicated across nodes, providing resilience against node failures.

The current implementation, Hazelcast was successfully configured to distribute data storage across multiple machines. The system uses a HazelcastIndexerNode on the main computer and HazelcastWorkerNodes on additional computers to form a distributed computing cluster.

## System Testing

System testing was performed on multiple computers within the same network. In the current setup, two computers were used, each running a single instance of the JavaQuery module. NGINX distributed incoming requests between these instances.

## Conclusion

NGINX's load balancing capabilities, combined with Hazelcast for distributed data storage, provide a robust solution for scalable and efficient distributed systems. This approach ensures even resource utilization while offering flexibility for future expansions.
