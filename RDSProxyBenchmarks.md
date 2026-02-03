# RDS Proxy Benchmarks

## Agenda

- Introduction to RDS Proxy
- Brief Introduction to RDS and Connection Limitations
- How RDS Proxy Solves Connection Issues
- Creating an RDS Proxy: Considerations and Limitations
- Benchmark Comparison: MySQLslap with 500 Concurrent Connections

## Introduction to RDS Proxy

Amazon RDS Proxy is a fully managed, highly available database proxy for Amazon Relational Database Service (RDS) that makes applications more scalable, more resilient to database failures, and more secure. It acts as an intermediary between your application and the database, managing connections efficiently and providing features like connection pooling, failover handling, and IAM authentication.

## Brief Introduction to RDS and Connection Limitations

Amazon Relational Database Service (RDS) is a managed relational database service that simplifies the setup, operation, and scaling of databases in the cloud. It supports popular engines like MySQL, PostgreSQL, MariaDB, Oracle, and SQL Server.

However, RDS has limitations when handling multiple connections, especially in high-traffic scenarios. For instance, We were once running two services (Laravel PHP) with expected peak traffic of 200K (maxed to almost 1.1M requests) using ECS (Elastic Container Service) environment. The instant surge in traffic (from 0 to 100k users) resulted in the ECS scaling up and spinning new tasks (instances of your containers) which would initially test database connectivity before starting the application and becoming healthy. Since we used RDS endpoints directly, the tasks would fail the health status checks as some of the tasks could not establish connection with the database. This resulted in us increasing the instance size (metrics showed CPU maxing out) and adding more read replicas, basically *fighting bad design with more infrastructure* to compensate for inefficient connection management. Each connection consumes database resources, and without proper pooling, this can overwhelm the database, leading to performance degradation, timeouts, and increased costs.

## How RDS Proxy Solves Connection Issues

RDS Proxy addresses these limitations by implementing connection multiplexing and pooling. Instead of each application connection directly hitting the database, the proxy maintains a pool of persistent connections to the database and multiplexes application requests over these connections. This reduces the number of actual database connections, allowing for better resource utilization and scalability.

Key benefits include:
- **Connection Pooling**: Reuses connections efficiently, reducing overhead.
- **Failover Handling**: Automatically routes traffic to a healthy database instance during failovers.
- **Security**: Supports IAM authentication and integrates with AWS Secrets Manager.
- **Scalability**: Handles sudden spikes in traffic without exhausting database connections.

In the ECS scenario mentioned, RDS Proxy can prevent the need for over-provisioning instances by managing connections more effectively, leading to cost savings and improved performance.

## Creating an RDS Proxy: Considerations and Limitations

### Creating an RDS Proxy

To create an RDS Proxy:

1. **Prerequisites**:
   - An existing RDS DB instance or cluster.
   - VPC and subnets configured.
   - Optionally, AWS Secrets Manager for credentials.

2. **Steps**:
   - Navigate to the RDS console.
   - Select "Proxies" and create a new proxy.
   - Choose the target database (RDS instance or Aurora cluster).
   - Configure connection settings, such as maximum connections and idle timeout.
   - Set up authentication (IAM or Secrets Manager).
   - Associate with a VPC and security groups.

### Considerations

- **Engine Compatibility**: Supports MySQL, PostgreSQL, and MariaDB.
- **VPC Requirements**: Must be in the same VPC as the database.
- **Cost**: Additional charges based on proxy size and connections.
- **Monitoring**: Use CloudWatch for metrics like connection counts and latency.

### Limitations

- **Not for All Engines**: Does not support Oracle or SQL Server.
- **Latency**: Slight increase due to proxy overhead.
- **Features**: Some advanced database features may not be fully supported through the proxy.
- **Region Availability**: Available in most AWS regions, but check for updates.

## Benchmark Comparison: MySQLslap with 500 Concurrent Connections

To compare performance, we ran benchmarks using `mysqlslap` with varying concurrent connections against both the direct RDS endpoint and the RDS Proxy endpoint. `mysqlslap` is part of the mysql-client tooling and doesn't necessarily need extra setup. Other tools can also be used to compare and run benchmarks.

### Test Setup

- **Tool**: mysqlslap
- **Concurrency**: target 500 connections
- **Queries**: Simple SELECT statement
- **Environment**: Same VPC, Same RDS instance, Same jumpserver running mysqlslap script.

### Results

#### Aurora MySQL t4g.large maximum connections hard limit

![Instance max connections is 135](/screenshots/rdsproxy/instance-max-connections.png)

#### RDS Endpoint with 200 concurrent connections

![Extra connections error](/screenshots/rdsproxy/rds-endpoint-too-many-connections.png)

![RDS Benchmark results](/screenshots/rdsproxy/rds-enpoint-200-connections.png)


#### RDS Proxy Endpoint with 200 concurrent connections

![RDS Proxy Benchmark with 200 connections](/screenshots/rdsproxy/rds-proxy-200-connections.png) 

#### RDS Proxy Endpoint with 500 concurrent connections

![RDS Proxy Benchmark with 500 connections](/screenshots/rdsproxy/rds-proxy-500-connections.png)


The hard limit for max_connections for our t4g.large instance as shown is 135. Basically, the database can gracefully handle upto 135 connections. However the moment we increase this to 200, we start seeing `Too many connection` errors for the surplus.   

The RDS Proxy does not drop connections. However, notice the average number of seconds is slightly higher as compared to when using the rds endpoint directly.
Even when we 5x the number of concurrent connections (500) and double the iteration, the proxy still successfully handles the connections. This demonstrates how the proxy can [benefit] applications under high concurrency as was the case with our ECS scenario.

## References
- [Amazon RDS Proxy Documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html)
- [Monitoring RDS Proxy](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.monitoring.html)
- [mysqlslap Documentation](https://dev.mysql.com/doc/refman/8.0/en/mysqlslap.html)
- [RDS Proxy in production](https://www.tothenew.com/blog/rds-proxy-in-production-real-world-lessons-limitations-and-why-we-use-it/)