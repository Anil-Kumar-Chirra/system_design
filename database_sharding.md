# Database Sharding and Partitioning

## Table of Contents
- [Overview](#overview)
- [Sharding vs Partitioning](#sharding-vs-partitioning)
- [Types of Partitioning](#types-of-partitioning)
- [Sharding Strategies](#sharding-strategies)
- [Implementation Approaches](#implementation-approaches)
- [Challenges and Solutions](#challenges-and-solutions)
- [Real-World Examples](#real-world-examples)
- [When to Shard](#when-to-shard)
- [Monitoring and Maintenance](#monitoring-and-maintenance)
- [Best Practices](#best-practices)
- [Trade-offs](#trade-offs)

## Overview

Database sharding and partitioning are techniques used to distribute data across multiple database instances or storage locations to improve performance, scalability, and availability. As applications grow, a single database often becomes a bottleneck, making these techniques essential for large-scale systems.

### Key Benefits
- **Improved Performance**: Parallel processing across multiple databases
- **Better Scalability**: Horizontal scaling by adding more database instances
- **Enhanced Availability**: Failure of one shard doesn't affect others
- **Cost Optimization**: Distribute expensive operations across cheaper hardware

## Sharding vs Partitioning

### Partitioning
- **Definition**: Dividing a large table into smaller, more manageable pieces within the same database instance
- **Scope**: Single database server
- **Management**: Handled by the database engine
- **Transparency**: Often transparent to the application

### Sharding
- **Definition**: Distributing data across multiple separate database instances
- **Scope**: Multiple database servers
- **Management**: Handled at the application level
- **Transparency**: Requires application-level logic

```
Partitioning:
┌─────────────────┐
│   Database      │
│ ┌─────┬─────┐   │
│ │Part1│Part2│   │
│ └─────┴─────┘   │
└─────────────────┘

Sharding:
┌─────────┐  ┌─────────┐  ┌─────────┐
│Database1│  │Database2│  │Database3│
│ Shard A │  │ Shard B │  │ Shard C │
└─────────┘  └─────────┘  └─────────┘
```

## Types of Partitioning

### 1. Horizontal Partitioning (Sharding)
Splits rows of a table across multiple databases based on a partitioning key.

**Example**: User table partitioned by user_id
```sql
-- Shard 1: user_id 1-1000
-- Shard 2: user_id 1001-2000
-- Shard 3: user_id 2001-3000
```

### 2. Vertical Partitioning
Splits columns of a table across multiple databases.

**Example**: User profile split by data type
```sql
-- Database 1: Basic Info
users_basic: user_id, username, email, created_at

-- Database 2: Profile Details  
users_profile: user_id, bio, avatar_url, preferences

-- Database 3: Analytics
users_analytics: user_id, last_login, login_count, activity_score
```

### 3. Functional Partitioning
Splits data based on feature or service boundaries.

**Example**: E-commerce platform
```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   Users DB  │  │ Products DB │  │  Orders DB  │
│             │  │             │  │             │
│ - users     │  │ - products  │  │ - orders    │
│ - profiles  │  │ - inventory │  │ - payments  │
│ - auth      │  │ - reviews   │  │ - shipping  │
└─────────────┘  └─────────────┘  └─────────────┘
```

## Sharding Strategies

### 1. Range-Based Sharding
Data is partitioned based on ranges of the sharding key.

```python
def get_shard_by_range(user_id):
    if 1 <= user_id <= 10000:
        return "shard_1"
    elif 10001 <= user_id <= 20000:
        return "shard_2"
    elif 20001 <= user_id <= 30000:
        return "shard_3"
    else:
        return "shard_overflow"
```

**Pros:**
- Simple to implement
- Range queries are efficient
- Easy to add new shards

**Cons:**
- Can create hotspots
- Uneven data distribution
- Sequential IDs can cause imbalanced load

### 2. Hash-Based Sharding
Uses a hash function to determine the shard.

```python
import hashlib

def get_shard_by_hash(user_id, num_shards=4):
    hash_value = int(hashlib.md5(str(user_id).encode()).hexdigest(), 16)
    return f"shard_{hash_value % num_shards + 1}"

# Example usage
user_id = 12345
shard = get_shard_by_hash(user_id)  # Returns: shard_2
```

**Pros:**
- Even data distribution
- No hotspots
- Simple to implement

**Cons:**
- Range queries are expensive
- Difficult to add/remove shards
- No locality for related data

### 3. Directory-Based Sharding
Uses a lookup service to determine the shard location.

```python
class ShardDirectory:
    def __init__(self):
        self.directory = {}
        self.shard_mapping = {
            "users_1_10000": "shard_1",
            "users_10001_20000": "shard_2",
            "orders_2023": "shard_3",
            "orders_2024": "shard_4"
        }
    
    def get_shard(self, table, key):
        lookup_key = f"{table}_{key}"
        # Complex logic to determine shard
        return self.determine_shard(lookup_key)
    
    def determine_shard(self, lookup_key):
        # Custom logic based on business rules
        pass
```

**Pros:**
- Flexible shard assignment
- Easy to migrate data
- Can optimize for query patterns

**Cons:**
- Additional complexity
- Directory service becomes a bottleneck
- Single point of failure

### 4. Consistent Hashing
Distributes data in a way that minimizes reorganization when adding/removing shards.

```python
import hashlib
import bisect

class ConsistentHash:
    def __init__(self, nodes=None, replicas=3):
        self.replicas = replicas
        self.ring = {}
        self.sorted_keys = []
        
        if nodes:
            for node in nodes:
                self.add_node(node)
    
    def add_node(self, node):
        for i in range(self.replicas):
            key = self.hash(f"{node}:{i}")
            self.ring[key] = node
            self.sorted_keys.append(key)
        self.sorted_keys.sort()
    
    def remove_node(self, node):
        for i in range(self.replicas):
            key = self.hash(f"{node}:{i}")
            del self.ring[key]
            self.sorted_keys.remove(key)
    
    def get_node(self, key):
        if not self.ring:
            return None
        hash_key = self.hash(key)
        idx = bisect.bisect_right(self.sorted_keys, hash_key)
        if idx == len(self.sorted_keys):
            idx = 0
        return self.ring[self.sorted_keys[idx]]
    
    def hash(self, key):
        return int(hashlib.md5(key.encode()).hexdigest(), 16)

# Example usage
ch = ConsistentHash(['shard_1', 'shard_2', 'shard_3'])
shard = ch.get_node('user_12345')
```

## Implementation Approaches

### 1. Application-Level Sharding
The application manages shard routing and data distribution.

```python
class ShardManager:
    def __init__(self):
        self.shards = {
            'shard_1': DatabaseConnection('db1.example.com'),
            'shard_2': DatabaseConnection('db2.example.com'),
            'shard_3': DatabaseConnection('db3.example.com'),
        }
    
    def get_connection(self, shard_key):
        shard_id = self.calculate_shard(shard_key)
        return self.shards[shard_id]
    
    def execute_query(self, shard_key, query, params):
        connection = self.get_connection(shard_key)
        return connection.execute(query, params)
    
    def execute_cross_shard_query(self, query, params):
        results = []
        for shard_id, connection in self.shards.items():
            result = connection.execute(query, params)
            results.extend(result)
        return results
```

### 2. Middleware/Proxy-Based Sharding
A proxy layer handles shard routing transparently.

```
Application
     │
     ▼
┌─────────────┐
│Sharding Proxy│  (Vitess, ProxySQL, etc.)
└─────────────┘
     │
     ▼
┌─────┬─────┬─────┐
│Shard│Shard│Shard│
│  1  │  2  │  3  │
└─────┴─────┴─────┘
```

### 3. Database-Native Sharding
The database system handles sharding automatically.

Examples:
- **MongoDB**: Automatic sharding with configurable shard keys
- **PostgreSQL**: Table partitioning with pg_partman
- **MySQL**: Partitioning with PARTITION BY clauses

## Challenges and Solutions

### 1. Cross-Shard Queries
**Problem**: Queries that need data from multiple shards are expensive.

**Solutions:**
```python
# Denormalization approach
class UserOrderSummary:
    def __init__(self):
        # Store summary data in the same shard as user
        self.user_shard_data = {
            'user_id': 12345,
            'total_orders': 25,
            'last_order_date': '2024-01-15',
            'favorite_categories': ['electronics', 'books']
        }

# Scatter-Gather approach
async def get_user_orders_across_shards(user_id):
    tasks = []
    for shard in shards:
        task = shard.get_orders_by_user(user_id)
        tasks.append(task)
    
    results = await asyncio.gather(*tasks)
    return merge_results(results)
```

### 2. Hotspots and Uneven Distribution
**Problem**: Some shards receive more traffic than others.

**Solutions:**
- **Consistent Hashing**: Distribute load more evenly
- **Shard Splitting**: Split hot shards into smaller ones
- **Load Monitoring**: Track and rebalance based on metrics

```python
class HotspotDetector:
    def __init__(self, threshold=1000):
        self.request_counts = {}
        self.threshold = threshold
    
    def record_request(self, shard_id):
        self.request_counts[shard_id] = self.request_counts.get(shard_id, 0) + 1
    
    def detect_hotspots(self):
        avg_requests = sum(self.request_counts.values()) / len(self.request_counts)
        hotspots = []
        
        for shard_id, count in self.request_counts.items():
            if count > avg_requests * 2:  # 200% of average
                hotspots.append(shard_id)
        
        return hotspots
```

### 3. Rebalancing and Resharding
**Problem**: Adding or removing shards requires data migration.

**Solutions:**
```python
class ShardRebalancer:
    def __init__(self, old_shards, new_shards):
        self.old_shards = old_shards
        self.new_shards = new_shards
    
    async def migrate_data(self):
        # 1. Stop writes to old shards
        await self.pause_writes()
        
        # 2. Copy data to new shards
        for old_shard in self.old_shards:
            data = await old_shard.export_data()
            target_shards = self.calculate_new_distribution(data)
            await self.import_to_shards(data, target_shards)
        
        # 3. Update routing configuration
        await self.update_shard_config()
        
        # 4. Resume writes
        await self.resume_writes()
```

### 4. Maintaining Referential Integrity
**Problem**: Foreign key relationships across shards are difficult to maintain.

**Solutions:**
- **Denormalization**: Store related data together
- **Application-level checks**: Validate relationships in code
- **Event-driven consistency**: Use events to maintain consistency

```python
class CrossShardReferenceManager:
    def __init__(self):
        self.reference_cache = {}
    
    async def create_order(self, user_id, product_id):
        # Validate user exists
        user_shard = self.get_user_shard(user_id)
        user = await user_shard.get_user(user_id)
        if not user:
            raise ValueError("User not found")
        
        # Validate product exists
        product_shard = self.get_product_shard(product_id)
        product = await product_shard.get_product(product_id)
        if not product:
            raise ValueError("Product not found")
        
        # Create order with denormalized data
        order_data = {
            'user_id': user_id,
            'user_email': user.email,  # Denormalized
            'product_id': product_id,
            'product_name': product.name,  # Denormalized
            'price': product.price
        }
        
        order_shard = self.get_order_shard(user_id)
        return await order_shard.create_order(order_data)
```

## Real-World Examples

### 1. Instagram - User Data Sharding
Instagram shards user data based on user ID using a consistent hashing approach.

```python
# Simplified Instagram-like user sharding
class InstagramSharding:
    def __init__(self):
        self.shards = ['db1', 'db2', 'db3', 'db4']
    
    def get_user_shard(self, user_id):
        return self.shards[user_id % len(self.shards)]
    
    def get_user_posts_shard(self, user_id):
        # Posts are stored in the same shard as user
        return self.get_user_shard(user_id)
    
    def get_feed_data(self, user_id, following_list):
        # Challenge: Getting posts from users across different shards
        user_posts = {}
        for followed_user_id in following_list:
            shard = self.get_user_shard(followed_user_id)
            posts = self.query_shard(shard, f"SELECT * FROM posts WHERE user_id = {followed_user_id}")
            user_posts[followed_user_id] = posts
        
        return self.merge_and_sort_by_time(user_posts)
```

### 2. Uber - Trip Data Partitioning
Uber partitions trip data by geographical regions and time.

```python
class UberSharding:
    def __init__(self):
        self.geographic_shards = {
            'us_west': 'shard_us_west',
            'us_east': 'shard_us_east',
            'europe': 'shard_europe',
            'asia': 'shard_asia'
        }
        self.time_partitions = ['2024_q1', '2024_q2', '2024_q3', '2024_q4']
    
    def get_trip_shard(self, lat, lng, timestamp):
        region = self.get_region(lat, lng)
        time_partition = self.get_time_partition(timestamp)
        return f"{self.geographic_shards[region]}_{time_partition}"
    
    def get_region(self, lat, lng):
        # Simplified region detection
        if -125 <= lng <= -66:  # US longitude range
            return 'us_west' if lng <= -95 else 'us_east'
        elif -10 <= lng <= 40:  # Europe longitude range
            return 'europe'
        else:
            return 'asia'
```

### 3. WhatsApp - Message Storage
WhatsApp shards messages based on chat ID to keep conversation data together.

```python
class WhatsAppSharding:
    def __init__(self):
        self.message_shards = 1000  # 1000 shards for messages
    
    def get_chat_shard(self, chat_id):
        # Keep all messages of a chat in the same shard
        return f"messages_shard_{hash(chat_id) % self.message_shards}"
    
    def store_message(self, chat_id, message):
        shard = self.get_chat_shard(chat_id)
        return self.write_to_shard(shard, {
            'chat_id': chat_id,
            'message_id': message.id,
            'sender_id': message.sender_id,
            'content': message.content,
            'timestamp': message.timestamp
        })
    
    def get_chat_messages(self, chat_id, limit=50):
        shard = self.get_chat_shard(chat_id)
        return self.query_shard(shard, 
            f"SELECT * FROM messages WHERE chat_id = '{chat_id}' "
            f"ORDER BY timestamp DESC LIMIT {limit}")
```

## When to Shard

### Consider Sharding When:
- **Database size > 500GB-1TB** per server
- **Query performance degrades** despite optimization
- **Write throughput** exceeds single server capacity
- **Storage costs** become significant
- **Backup/recovery time** is too long
- **Geographic distribution** is needed

### Don't Shard When:
- **Data fits comfortably** on a single server
- **Vertical scaling** is still possible and cost-effective
- **Cross-shard queries** are the majority of your workload
- **Team lacks expertise** in distributed systems
- **Application complexity** increase isn't justified

### Decision Framework:
```python
class ShardingDecisionFramework:
    def should_shard(self, metrics):
        score = 0
        
        # Database size factor
        if metrics['db_size_gb'] > 1000:
            score += 3
        elif metrics['db_size_gb'] > 500:
            score += 2
        elif metrics['db_size_gb'] > 200:
            score += 1
        
        # Performance factor
        if metrics['avg_query_time_ms'] > 1000:
            score += 3
        elif metrics['avg_query_time_ms'] > 500:
            score += 2
        
        # Growth factor
        if metrics['monthly_growth_rate'] > 0.5:  # 50% monthly growth
            score += 2
        elif metrics['monthly_growth_rate'] > 0.2:  # 20% monthly growth
            score += 1
        
        # Cross-shard query penalty
        if metrics['cross_shard_query_percentage'] > 0.7:  # 70% cross-shard
            score -= 3
        elif metrics['cross_shard_query_percentage'] > 0.5:  # 50% cross-shard
            score -= 2
        
        return score >= 5  # Threshold for sharding recommendation
```

## Monitoring and Maintenance

### Key Metrics to Monitor:
```python
class ShardingMetrics:
    def __init__(self):
        self.metrics = {}
    
    def collect_shard_metrics(self):
        for shard_id in self.shards:
            metrics = {
                'query_latency': self.get_avg_latency(shard_id),
                'query_throughput': self.get_qps(shard_id),
                'storage_usage': self.get_storage_usage(shard_id),
                'connection_count': self.get_active_connections(shard_id),
                'error_rate': self.get_error_rate(shard_id),
                'replication_lag': self.get_replication_lag(shard_id)
            }
            self.metrics[shard_id] = metrics
    
    def detect_issues(self):
        issues = []
        for shard_id, metrics in self.metrics.items():
            if metrics['query_latency'] > 1000:  # 1 second
                issues.append(f"High latency on {shard_id}")
            
            if metrics['storage_usage'] > 0.8:  # 80% full
                issues.append(f"Storage almost full on {shard_id}")
            
            if metrics['error_rate'] > 0.05:  # 5% error rate
                issues.append(f"High error rate on {shard_id}")
        
        return issues
```

### Automated Maintenance Tasks:
```python
class ShardMaintenance:
    def __init__(self):
        self.scheduler = TaskScheduler()
    
    def setup_maintenance_tasks(self):
        # Daily tasks
        self.scheduler.daily(self.collect_shard_statistics)
        self.scheduler.daily(self.check_shard_balance)
        self.scheduler.daily(self.verify_data_integrity)
        
        # Weekly tasks
        self.scheduler.weekly(self.analyze_query_patterns)
        self.scheduler.weekly(self.review_shard_performance)
        
        # Monthly tasks
        self.scheduler.monthly(self.plan_capacity_expansion)
        self.scheduler.monthly(self.evaluate_resharding_needs)
    
    async def check_shard_balance(self):
        metrics = await self.get_all_shard_metrics()
        imbalance = self.calculate_imbalance(metrics)
        
        if imbalance > 0.3:  # 30% imbalance threshold
            await self.trigger_rebalancing_alert()
```

## Best Practices

### 1. Choose the Right Shard Key
```python
# Good shard keys
good_shard_keys = [
    'user_id',      # High cardinality, even distribution
    'tenant_id',    # Natural isolation boundary
    'region_id',    # Geographic distribution
    'timestamp'     # Time-based partitioning
]

# Bad shard keys
bad_shard_keys = [
    'status',       # Low cardinality (active/inactive)
    'created_date', # May create hotspots on current date
    'category',     # Uneven distribution likely
    'boolean_flag'  # Only 2 possible values
]

def evaluate_shard_key(column_name, sample_data):
    cardinality = len(set(sample_data[column_name]))
    total_rows = len(sample_data)
    
    # Calculate distribution evenness
    value_counts = {}
    for value in sample_data[column_name]:
        value_counts[value] = value_counts.get(value, 0) + 1
    
    max_count = max(value_counts.values())
    min_count = min(value_counts.values())
    
    evenness_score = min_count / max_count if max_count > 0 else 0
    cardinality_score = min(cardinality / total_rows, 1.0)
    
    overall_score = (evenness_score + cardinality_score) / 2
    
    return {
        'score': overall_score,
        'recommendation': 'Good' if overall_score > 0.7 else 'Consider alternatives'
    }
```

### 2. Design for Query Patterns
```python
class QueryPatternOptimizer:
    def __init__(self):
        self.query_patterns = {}
    
    def analyze_queries(self, query_log):
        patterns = {
            'single_shard': 0,
            'cross_shard': 0,
            'broadcast': 0
        }
        
        for query in query_log:
            pattern_type = self.classify_query(query)
            patterns[pattern_type] += 1
        
        return patterns
    
    def optimize_shard_design(self, patterns):
        recommendations = []
        
        if patterns['cross_shard'] / sum(patterns.values()) > 0.3:
            recommendations.append("Consider denormalization to reduce cross-shard queries")
        
        if patterns['broadcast'] / sum(patterns.values()) > 0.1:
            recommendations.append("Consider read replicas for broadcast queries")
        
        return recommendations
```

### 3. Implement Gradual Migration
```python
class GradualMigration:
    def __init__(self, old_db, new_shards):
        self.old_db = old_db
        self.new_shards = new_shards
        self.migration_percentage = 0
    
    async def migrate_gradually(self, batch_size=1000):
        while self.migration_percentage < 100:
            # Read batch from old database
            batch = await self.old_db.get_next_batch(batch_size)
            
            # Write to new shards
            for record in batch:
                shard = self.determine_shard(record)
                await shard.insert(record)
            
            # Update migration progress
            self.migration_percentage += (batch_size / self.total_records) * 100
            
            # Verify data integrity
            await self.verify_batch_integrity(batch)
            
            # Allow for pause/resume
            await self.check_for_pause_signal()
    
    async def dual_write_phase(self, record):
        # Write to both old and new systems during transition
        await self.old_db.write(record)
        
        shard = self.determine_shard(record)
        await shard.write(record)
        
        # Compare results for consistency
        await self.verify_consistency(record)
```

## Trade-offs

### Performance vs Complexity
| Aspect | Single Database | Sharded Database |
|--------|----------------|------------------|
| **Query Latency** | Fast for small datasets | Consistent performance at scale |
| **Cross-table Joins** | Fast and simple | Complex and slower |
| **Transactions** | ACID guaranteed | Limited cross-shard transactions |
| **Development Complexity** | Simple | High complexity |
| **Operational Overhead** | Low | High |
| **Debugging** | Straightforward | Complex distributed debugging |

### Consistency vs Availability
```python
# Strong consistency (slower)
class StrongConsistencySharding:
    async def write_with_strong_consistency(self, data):
        # Two-phase commit across shards
        transaction_id = self.generate_transaction_id()
        
        # Phase 1: Prepare
        prepare_results = []
        for shard in self.affected_shards(data):
            result = await shard.prepare(transaction_id, data)
            prepare_results.append(result)
        
        # Phase 2: Commit or abort
        if all(result.success for result in prepare_results):
            for shard in self.affected_shards(data):
                await shard.commit(transaction_id)
        else:
            for shard in self.affected_shards(data):
                await shard.abort(transaction_id)

# Eventual consistency (faster)
class EventualConsistencySharding:
    async def write_with_eventual_consistency(self, data):
        # Write to primary shard immediately
        primary_shard = self.get_primary_shard(data)
        await primary_shard.write(data)
        
        # Async replication to other shards
        for shard in self.get_replica_shards(data):
            self.background_tasks.add_task(shard.replicate, data)
```

### Cost vs Performance
- **Horizontal Scaling**: More servers = higher costs but better performance
- **Vertical Scaling**: Fewer, more powerful servers = potentially lower costs but limited scalability
- **Hybrid Approach**: Combination of both based on workload characteristics

---

## Conclusion

Database sharding and partitioning are powerful techniques for scaling applications, but they come with significant complexity. The decision to shard should be based on clear performance requirements, growth projections, and team capabilities.

### Key Takeaways:
1. **Start Simple**: Use vertical scaling and optimization before sharding
2. **Choose Shard Keys Carefully**: This decision impacts everything else
3. **Design for Your Query Patterns**: Minimize cross-shard operations
4. **Monitor Continuously**: Detect issues before they impact users
5. **Plan for Migration**: Resharding is often necessary as you grow
6. **Consider Alternatives**: Sometimes denormalization or caching is better than sharding

### Next Steps:
- Practice implementing different sharding strategies
- Study real-world case studies from companies like Netflix, Uber, and Discord
- Experiment with database-native sharding features
- Learn about related topics like consistent hashing and distributed consensus