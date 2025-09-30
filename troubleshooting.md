# Cortex Analyst Troubleshooting Guide

This comprehensive troubleshooting guide addresses common issues encountered when implementing and using Cortex Analyst, with step-by-step solutions based on official Snowflake documentation.

## Table of Contents
1. [Semantic Model Setup Issues](#semantic-model-setup-issues)
2. [Verified Query Repository Problems](#verified-query-repository-problems)
3. [Cortex Search Integration Issues](#cortex-search-integration-issues)
4. [Monitoring and Logging Problems](#monitoring-and-logging-problems)
5. [Query Performance Issues](#query-performance-issues)
6. [API Integration Problems](#api-integration-problems)
7. [User Experience Issues](#user-experience-issues)
8. [Data Quality and Accuracy Problems](#data-quality-and-accuracy-problems)

---

## Semantic Model Setup Issues

### Issue 1: Cannot Create Semantic Model

#### Symptoms
- "Permission denied" errors when creating semantic models
- Model creation fails silently
- Cannot access semantic model creation interface

#### Root Causes & Solutions

**Cause 1: Insufficient Permissions**
```sql
-- Check current role permissions
SHOW GRANTS TO ROLE current_role();

-- Required permissions for semantic model creation
GRANT USAGE ON DATABASE your_database TO ROLE your_role;
GRANT USAGE ON SCHEMA your_database.your_schema TO ROLE your_role;
GRANT SELECT ON ALL TABLES IN SCHEMA your_database.your_schema TO ROLE your_role;
GRANT CREATE VIEW ON SCHEMA your_database.your_schema TO ROLE your_role;

-- For warehouse usage
GRANT USAGE ON WAREHOUSE your_warehouse TO ROLE your_role;
```

**Cause 2: Missing Database/Schema Objects**
```sql
-- Verify database and schema exist
SHOW DATABASES LIKE 'your_database%';
SHOW SCHEMAS IN DATABASE your_database;

-- Verify table accessibility
SELECT * FROM your_database.your_schema.your_table LIMIT 1;
```

**Cause 3: Warehouse Issues**
```sql
-- Check warehouse status
SHOW WAREHOUSES LIKE 'your_warehouse%';

-- Ensure warehouse is running
ALTER WAREHOUSE your_warehouse RESUME;

-- Check warehouse permissions
SHOW GRANTS ON WAREHOUSE your_warehouse;
```

### Issue 2: Tables Not Appearing in Model Builder

#### Symptoms
- Empty table list in semantic model creation UI
- Expected tables missing from selection
- "No tables found" error

#### Diagnostic Steps
```sql
-- 1. Verify table existence and permissions
SHOW TABLES IN SCHEMA your_database.your_schema;

-- 2. Test direct table access
SELECT COUNT(*) FROM your_database.your_schema.your_table;

-- 3. Check current role and context
SELECT CURRENT_ROLE(), CURRENT_DATABASE(), CURRENT_SCHEMA();

-- 4. Verify grants on specific table
SHOW GRANTS ON TABLE your_database.your_schema.your_table;
```

#### Solutions
```sql
-- Solution 1: Grant explicit table permissions
GRANT SELECT ON TABLE your_database.your_schema.your_table TO ROLE your_role;

-- Solution 2: Set proper database context
USE DATABASE your_database;
USE SCHEMA your_schema;

-- Solution 3: Check for case sensitivity issues
SHOW TABLES LIKE 'YOUR_TABLE%';  -- Try uppercase
SHOW TABLES LIKE 'your_table%';  -- Try lowercase
```

### Issue 3: Dimension vs Fact Misclassification

#### Symptoms
- ID columns classified as facts instead of dimensions
- Numeric dimensions appearing as facts
- Unable to create relationships due to incorrect classification

#### Identification and Correction
```yaml
# ❌ Common misclassifications
# These are incorrectly auto-classified as facts:
facts:
  - name: customer_id        # Should be dimension
  - name: product_id         # Should be dimension  
  - name: order_date         # Should be dimension
  - name: category_code      # Should be dimension

# ✅ Correct classification
dimensions:
  - name: customer_id
    expr: customer_id
    unique_identifier: true
    description: "Unique customer identifier"
  
  - name: product_id
    expr: product_id
    unique_identifier: true
    description: "Unique product identifier"

facts:
  - name: order_amount
    expr: order_amount
    description: "Total order value in USD"
  
  - name: quantity
    expr: quantity
    description: "Number of items ordered"
```

#### Step-by-Step Correction Process
1. **Review Auto-Classification Results**
   - Check all numeric columns carefully
   - Verify business meaning of each field
   - Identify ID fields, dates, and categorical data

2. **Move Misclassified Fields**
   ```yaml
   # Use the semantic model editor to:
   # 1. Select the misclassified field
   # 2. Move from facts to dimensions (or vice versa)
   # 3. Set unique_identifier flag for primary keys
   # 4. Save the model
   ```

3. **Validate Changes**
   ```sql
   -- Test that relationships can now be created
   -- Verify queries work as expected
   ```

### Issue 4: Relationship Creation Failures

#### Symptoms
- Cannot see join columns in relationship setup
- "Invalid relationship" errors
- Relationships don't work in queries

#### Diagnostic Checklist
```yaml
# Verify these requirements:
# 1. Both tables included in semantic model ✓
# 2. Join columns marked as dimensions ✓
# 3. Primary key marked as unique_identifier ✓
# 4. Data types compatible ✓
# 5. Referential integrity in data ✓
```

#### Solutions

**Problem: Missing Unique Identifiers**
```yaml
# ✅ Fix: Mark primary keys as unique
dimensions:
  - name: customer_id
    expr: customer_id
    unique_identifier: true  # Required for relationships
    description: "Primary key for customers table"
```

**Problem: Data Type Mismatches**
```sql
-- Check data types of join columns
DESCRIBE TABLE customers;
DESCRIBE TABLE orders;

-- Ensure compatible types
-- Both should be NUMBER, or both VARCHAR, etc.
```

**Problem: Missing Referential Integrity**
```sql
-- Check for orphaned records
SELECT COUNT(*) 
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;

-- Clean up orphaned records if found
DELETE FROM orders 
WHERE customer_id NOT IN (SELECT customer_id FROM customers);
```

---

## Verified Query Repository Problems

Based on [official VQR documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst/verified-query-repository), here are common VQR issues and solutions.

### Issue 5: VQR Queries Not Working

#### Symptoms
- Verified queries return different results than expected
- VQR queries fail with errors
- Inconsistent behavior between verified and new queries

#### Diagnostic Process
```sql
-- 1. Check VQR query status in monitoring
SELECT 
  user_question,
  error_message,
  generated_sql,
  request_timestamp
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW', 
  'your_database.your_schema.your_semantic_view'
))
WHERE user_question IN (
  'your verified query here',
  'another verified query'
)
ORDER BY request_timestamp DESC
LIMIT 10;
```

#### Solutions

**Problem: Outdated VQR Entries**
```yaml
# Remove or update outdated verified queries
verified_queries:
  # ❌ Remove if no longer applicable
  # - question: "Show Q3 2023 sales"  # Outdated date reference
  #   verified_at: "2023-10-15T10:30:00Z"
  
  # ✅ Keep current and relevant
  - question: "Show last quarter's sales"
    verified_at: "2024-01-15T10:30:00Z"
```

**Problem: VQR Schema Changes**
```yaml
# Update VQR when semantic model changes
verified_queries:
  # If you renamed a dimension, update VQR accordingly
  - question: "Show revenue by customer segment"  # Updated terminology
    verified_at: "2024-01-15T10:30:00Z"
  
  # Remove VQR for deleted fields
  # - question: "Show sales by old_field_name"  # Field no longer exists
```

### Issue 6: VQR Performance Degradation

#### Symptoms
- VQR queries running slowly
- Timeout errors on previously fast queries
- Inconsistent response times

#### Performance Analysis
```sql
-- Analyze VQR query performance trends
WITH vqr_performance AS (
  SELECT 
    user_question,
    request_timestamp,
    generated_sql,
    LENGTH(generated_sql) as sql_complexity,
    ROW_NUMBER() OVER (PARTITION BY user_question ORDER BY request_timestamp DESC) as rn
  FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
    'SEMANTIC_VIEW', 
    'your_database.your_schema.your_semantic_view'
  ))
  WHERE user_question IN (
    -- List your verified queries here
    'What was our total revenue last month?',
    'Which products are our top sellers?'
  )
  AND request_timestamp >= DATEADD('day', -30, CURRENT_TIMESTAMP())
)
SELECT 
  user_question,
  sql_complexity,
  COUNT(*) as query_frequency
FROM vqr_performance
WHERE rn = 1  -- Latest version of each query
GROUP BY user_question, sql_complexity
ORDER BY sql_complexity DESC;
```

#### Solutions

**Optimization 1: Create Summary Tables**
```sql
-- Create pre-aggregated tables for VQR queries
CREATE OR REPLACE TABLE summary_monthly_revenue AS
SELECT 
  DATE_TRUNC('month', order_date) as month,
  SUM(order_total) as total_revenue,
  COUNT(*) as order_count,
  COUNT(DISTINCT customer_id) as unique_customers
FROM orders
WHERE order_date >= DATEADD('year', -2, CURRENT_DATE())
GROUP BY DATE_TRUNC('month', order_date);

-- Update semantic model to use summary table
```

**Optimization 2: Add Appropriate Indexes**
```sql
-- Create indexes for frequently joined columns
CREATE INDEX IF NOT EXISTS idx_orders_customer_date 
  ON orders (customer_id, order_date);

CREATE INDEX IF NOT EXISTS idx_orders_product_date
  ON order_items (product_id, order_date);
```

---

## Cortex Search Integration Issues

Based on [official Cortex Search documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst/cortex-analyst-search-integration), here are common integration problems.

### Issue 7: Cortex Search Service Creation Failures

#### Symptoms
- "Failed to create search service" errors
- Search service creation times out
- Search service shows as failed status

#### Diagnostic Steps
```sql
-- Check search service status
SHOW CORTEX SEARCH SERVICES;

-- Check specific service details
DESCRIBE CORTEX SEARCH SERVICE your_search_service;

-- Verify source data
SELECT COUNT(*), COUNT(DISTINCT your_column) 
FROM your_table 
WHERE your_column IS NOT NULL;
```

#### Solutions

**Problem: Insufficient Warehouse Resources**
```sql
-- Use appropriately sized warehouse
ALTER CORTEX SEARCH SERVICE your_search_service 
  SET WAREHOUSE = 'MEDIUM';  -- Increase from XSMALL if needed

-- Check warehouse availability
SHOW WAREHOUSES LIKE 'your_warehouse%';
ALTER WAREHOUSE your_warehouse RESUME;
```

**Problem: Data Quality Issues**
```sql
-- Check for problematic data
SELECT 
  your_column,
  LENGTH(your_column) as text_length,
  CASE WHEN your_column REGEXP '[\x00-\x1F\x7F]' THEN 'Contains control chars' 
       WHEN LENGTH(your_column) > 1000 THEN 'Too long'
       ELSE 'OK' END as data_quality
FROM your_table
WHERE your_column IS NOT NULL
  AND (LENGTH(your_column) > 1000 OR your_column REGEXP '[\x00-\x1F\x7F]')
LIMIT 100;

-- Clean problematic data
CREATE OR REPLACE VIEW clean_data_for_search AS
SELECT 
  REGEXP_REPLACE(
    SUBSTR(your_column, 1, 1000),  -- Truncate long strings
    '[\x00-\x1F\x7F]',             -- Remove control characters
    ' '
  ) as your_column
FROM your_table
WHERE your_column IS NOT NULL
  AND LENGTH(TRIM(your_column)) > 0;
```

### Issue 8: Search Results Not Improving Queries

#### Symptoms
- Search integration enabled but literal matching still poor
- Users still can't find products/entities with natural language
- No improvement in query success rate

#### Verification Steps
```sql
-- Test search service directly
SELECT 
  PARSE_JSON(result):content::STRING as matched_text,
  PARSE_JSON(result):score::FLOAT as relevance_score
FROM TABLE(
  your_database.your_schema.your_search_service(
    'search query here'
  )
)
ORDER BY relevance_score DESC
LIMIT 10;
```

#### Troubleshooting Process

**Step 1: Verify Search Service Integration**
```yaml
# Check semantic model configuration
dimensions:
  - name: product_name
    expr: product_name
    description: "Product names for search"
    cortex_search_service:
      service: product_search_service       # ✓ Service name correct?
      literal_column: product_name          # ✓ Column name correct?
      database: your_database               # ✓ Database correct?
      schema: your_schema                   # ✓ Schema correct?
```

**Step 2: Test Search Quality**
```sql
-- Test various search terms
SELECT 'iphone' as search_term
UNION ALL SELECT 'apple phone'
UNION ALL SELECT 'cell phone'
UNION ALL SELECT 'smartphone';

-- Run each through search service
SELECT 
  'iphone' as search_term,
  PARSE_JSON(result):content::STRING as matched_product
FROM TABLE(your_database.your_schema.product_search_service('iphone'))
LIMIT 5;
```

**Step 3: Improve Search Data**
```sql
-- Enhance search corpus with synonyms and variations
CREATE OR REPLACE CORTEX SEARCH SERVICE enhanced_product_search
  ON product_search_text
  WAREHOUSE = small
  TARGET_LAG = '1 hour'
  AS (
    SELECT DISTINCT
      product_id,
      product_name as product_search_text,
      -- Include variations and synonyms
      category || ' ' || subcategory as category_text,
      brand || ' ' || product_name as brand_product_text
    FROM products
    WHERE product_name IS NOT NULL
    
    UNION ALL
    
    -- Add common synonyms
    SELECT 
      product_id,
      CASE 
        WHEN product_name ILIKE '%iphone%' THEN product_name || ' apple phone smartphone'
        WHEN product_name ILIKE '%samsung%' THEN product_name || ' android phone'
        ELSE product_name
      END as product_search_text,
      category || ' ' || subcategory as category_text,
      brand || ' ' || product_name as brand_product_text
    FROM products
    WHERE product_name IS NOT NULL
  );
```

---

## Monitoring and Logging Problems

Based on [official monitoring documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst/admin-observability), here are monitoring-related issues.

### Issue 9: Cannot Access Monitoring Logs

#### Symptoms
- Empty results from `CORTEX_ANALYST_REQUESTS` table function
- "Permission denied" when querying logs
- Monitoring views show no data

#### Permission Requirements Check
```sql
-- Verify required permissions
-- For semantic views:
SHOW GRANTS ON VIEW your_database.your_schema.your_semantic_view;

-- For file-based models:
SHOW GRANTS ON STAGE your_database.your_schema.your_stage;

-- Check if user has CORTEX_ANALYST_REQUESTS_ADMIN role
SHOW GRANTS TO USER current_user();
```

#### Solutions

**Solution 1: Grant Proper Permissions**
```sql
-- For semantic view monitoring
GRANT OWNERSHIP ON VIEW your_database.your_schema.your_semantic_view TO ROLE your_role;

-- Or at minimum, SELECT privilege on referenced tables
GRANT SELECT ON ALL TABLES IN SCHEMA your_database.your_schema TO ROLE your_role;

-- For stage-based models
GRANT WRITE ON STAGE your_database.your_schema.your_stage TO ROLE your_role;
```

**Solution 2: Use Admin Role for Organization-Wide Monitoring**
```sql
-- Request CORTEX_ANALYST_REQUESTS_ADMIN role from admin
-- This allows access to CORTEX_ANALYST_REQUESTS_V view

-- Query all requests across all models
SELECT 
  semantic_model_name,
  user_question,
  error_message,
  request_timestamp
FROM SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS_V
WHERE request_timestamp >= DATEADD('day', -7, CURRENT_TIMESTAMP())
ORDER BY request_timestamp DESC;
```

### Issue 10: Monitoring Data Lag

#### Symptoms
- Recent queries not appearing in monitoring logs
- 1-2 minute delay in log availability (expected)
- Missing data for certain time periods

#### Diagnostic Queries
```sql
-- Check latest log entry timestamp
SELECT MAX(request_timestamp) as latest_log_entry
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW', 
  'your_database.your_schema.your_semantic_view'
));

-- Compare with current time
SELECT 
  MAX(request_timestamp) as latest_log,
  CURRENT_TIMESTAMP() as current_time,
  DATEDIFF('minute', MAX(request_timestamp), CURRENT_TIMESTAMP()) as lag_minutes
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW', 
  'your_database.your_schema.your_semantic_view'
));
```

#### Understanding Normal vs Abnormal Lag
```sql
-- Normal: 1-2 minute lag
-- Abnormal: >5 minute lag may indicate issues

-- Check for gaps in data
SELECT 
  DATE_TRUNC('hour', request_timestamp) as hour_bucket,
  COUNT(*) as request_count
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW', 
  'your_database.your_schema.your_semantic_view'
))
WHERE request_timestamp >= DATEADD('day', -1, CURRENT_TIMESTAMP())
GROUP BY DATE_TRUNC('hour', request_timestamp)
ORDER BY hour_bucket;
```

---

## Query Performance Issues

### Issue 11: Slow Query Response Times

#### Symptoms
- Queries taking longer than 30 seconds
- Timeout errors
- High warehouse usage

#### Performance Analysis
```sql
-- Analyze query complexity and performance
SELECT 
  user_question,
  LENGTH(generated_sql) as sql_complexity,
  CASE 
    WHEN LENGTH(generated_sql) < 500 THEN 'Simple'
    WHEN LENGTH(generated_sql) < 1500 THEN 'Medium'
    ELSE 'Complex'
  END as complexity_category,
  COUNT(*) as frequency
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW', 
  'your_database.your_schema.your_semantic_view'
))
WHERE request_timestamp >= DATEADD('day', -7, CURRENT_TIMESTAMP())
  AND error_message IS NULL
GROUP BY user_question, LENGTH(generated_sql)
ORDER BY LENGTH(generated_sql) DESC;
```

#### Optimization Strategies

**Strategy 1: Warehouse Right-Sizing**
```sql
-- Monitor warehouse usage patterns
SELECT 
  warehouse_name,
  AVG(avg_running) as avg_queries_running,
  AVG(avg_queued_load) as avg_queued,
  SUM(credits_used) as total_credits
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_LOAD_HISTORY
WHERE start_time >= DATEADD('day', -7, CURRENT_TIMESTAMP())
GROUP BY warehouse_name;

-- Resize if consistently under-utilized or over-loaded
ALTER WAREHOUSE cortex_analyst_wh SET WAREHOUSE_SIZE = 'SMALL';  -- Scale up
-- ALTER WAREHOUSE cortex_analyst_wh SET WAREHOUSE_SIZE = 'XSMALL';  -- Scale down
```

**Strategy 2: Create Performance-Optimized Views**
```sql
-- Create materialized views for common aggregations
CREATE MATERIALIZED VIEW mv_monthly_sales AS
SELECT 
  DATE_TRUNC('month', order_date) as order_month,
  customer_segment,
  product_category,
  SUM(order_total) as total_revenue,
  COUNT(*) as order_count,
  COUNT(DISTINCT customer_id) as unique_customers
FROM orders o
JOIN customers c USING (customer_id)
JOIN order_items oi USING (order_id)
JOIN products p USING (product_id)
WHERE order_date >= DATEADD('year', -3, CURRENT_DATE())
GROUP BY 1, 2, 3;

-- Update semantic model to use materialized view for time-based queries
```

**Strategy 3: Query Pattern Optimization**
```sql
-- Identify most expensive query patterns
SELECT 
  user_question,
  generated_sql,
  COUNT(*) as frequency
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW', 
  'your_database.your_schema.your_semantic_view'
))
WHERE request_timestamp >= DATEADD('day', -7, CURRENT_TIMESTAMP())
  AND error_message IS NULL
  AND LENGTH(generated_sql) > 1000  -- Focus on complex queries
GROUP BY user_question, generated_sql
ORDER BY frequency DESC;

-- Optimize underlying data model for these patterns
```

### Issue 12: Memory and Resource Errors

#### Symptoms
- "Out of memory" errors
- "Result set too large" errors
- Query cancellation due to resource limits

#### Solutions

**Solution 1: Result Set Limitation**
```yaml
# Add custom instructions to limit result sizes
custom_instructions: |
  Query Limitations:
  - Limit all results to maximum 10,000 rows
  - For large datasets, show top 100 items by default
  - Use aggregation instead of detailed records when possible
  - Include appropriate filters for large time ranges
```

**Solution 2: Data Partitioning Strategy**
```sql
-- Implement table clustering for better performance
ALTER TABLE orders CLUSTER BY (order_date, customer_id);

-- Create partitioned views for large datasets
CREATE OR REPLACE VIEW recent_orders AS
SELECT * FROM orders 
WHERE order_date >= DATEADD('year', -1, CURRENT_DATE());

-- Update semantic model to use partitioned view
```

---

## API Integration Problems

### Issue 13: API Authentication Failures

#### Symptoms
- 401 "Unauthorized" errors
- Token expiration errors
- "Invalid credentials" messages

#### Authentication Troubleshooting
```python
import requests
import json

def test_api_authentication(account_url, token):
    """Test basic API connectivity"""
    headers = {
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json"
    }
    
    # Test with simple request
    test_url = f"{account_url}/api/v2/cortex/analyst/message"
    
    try:
        response = requests.get(test_url, headers=headers, timeout=10)
        print(f"Status Code: {response.status_code}")
        print(f"Response Headers: {dict(response.headers)}")
        
        if response.status_code == 401:
            print("❌ Authentication failed - check token")
        elif response.status_code == 404:
            print("❌ Endpoint not found - check URL")
        else:
            print("✅ Authentication successful")
            
    except requests.exceptions.RequestException as e:
        print(f"❌ Connection error: {e}")

# Usage
test_api_authentication(
    "https://your-account.snowflakecomputing.com",
    "your-token-here"
)
```

#### Token Management Best Practices
```python
import jwt
from datetime import datetime

def check_token_expiration(token):
    """Check if JWT token is expired"""
    try:
        # Decode without verification to check expiration
        decoded = jwt.decode(token, options={"verify_signature": False})
        exp_timestamp = decoded.get('exp')
        
        if exp_timestamp:
            exp_date = datetime.fromtimestamp(exp_timestamp)
            now = datetime.now()
            
            if exp_date < now:
                print(f"❌ Token expired on {exp_date}")
                return False
            else:
                print(f"✅ Token valid until {exp_date}")
                return True
        else:
            print("⚠️ No expiration found in token")
            return True
            
    except jwt.InvalidTokenError as e:
        print(f"❌ Invalid token format: {e}")
        return False

# Check your token
check_token_expiration("your-jwt-token-here")
```

### Issue 14: API Rate Limiting

#### Symptoms
- 429 "Too Many Requests" errors
- Requests failing during high usage
- Inconsistent API response times

#### Rate Limiting Handler
```python
import time
import random
from functools import wraps

def retry_with_backoff(max_retries=3, base_delay=1):
    """Decorator for handling rate limiting with exponential backoff"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    response = func(*args, **kwargs)
                    
                    if response.status_code == 429:
                        # Rate limited - wait before retry
                        delay = base_delay * (2 ** attempt) + random.uniform(0, 1)
                        print(f"Rate limited. Waiting {delay:.2f}s before retry {attempt + 1}")
                        time.sleep(delay)
                        continue
                    
                    return response
                    
                except requests.exceptions.RequestException as e:
                    if attempt == max_retries - 1:
                        raise e
                    time.sleep(base_delay * (2 ** attempt))
                    
            return None
        return wrapper
    return decorator

@retry_with_backoff(max_retries=3, base_delay=2)
def make_cortex_request(url, headers, payload):
    """Make API request with retry logic"""
    return requests.post(url, headers=headers, json=payload, timeout=30)

# Usage example
def query_with_retry(question, semantic_model):
    headers = {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}
    payload = {
        "messages": [{"role": "user", "content": [{"type": "text", "text": question}]}],
        "semantic_model": semantic_model
    }
    
    response = make_cortex_request(api_url, headers, payload)
    return response.json() if response else None
```

---

## User Experience Issues

### Issue 15: Poor Query Understanding

#### Symptoms
- Cortex Analyst doesn't understand business terms
- Queries return "I don't understand" responses
- Inconsistent interpretation of similar questions

#### Solutions

**Solution 1: Improve Synonyms and Descriptions**
```yaml
# ✅ Rich synonyms and descriptions
dimensions:
  - name: customer_segment
    expr: customer_segment
    description: "Customer classification based on annual spending and engagement. Premium customers spend >$5K annually, Standard customers spend $1K-$5K, Basic customers spend <$1K."
    synonyms: [
      "customer type", "customer tier", "customer category",
      "client segment", "customer classification", "customer level",
      "premium customers", "standard customers", "basic customers",
      "high value customers", "regular customers", "new customers"
    ]

facts:
  - name: monthly_recurring_revenue
    expr: monthly_recurring_revenue
    description: "Predictable monthly revenue from active subscriptions, excluding one-time payments, setup fees, and variable usage charges."
    synonyms: [
      "MRR", "recurring revenue", "subscription revenue",
      "monthly revenue", "subscription income", "recurring income",
      "monthly subscription fees", "regular monthly revenue"
    ]
```

**Solution 2: Add Business Context via Custom Instructions**
```yaml
custom_instructions: |
  Business Terminology Guide:
  
  ## Revenue Terms
  - "Sales" and "revenue" refer to total order value including tax
  - "Bookings" refer to committed future revenue from contracts
  - "ARR" (Annual Recurring Revenue) = MRR * 12
  
  ## Customer Terms  
  - "Customers" refers to unique individuals who have placed orders
  - "Accounts" refers to business entities (B2B context)
  - "Users" refers to people who log into the platform
  
  ## Product Terms
  - "SKU" and "product" are interchangeable
  - "Categories" are high-level groupings (Electronics, Clothing)
  - "Subcategories" are specific classifications within categories
  
  ## Time Periods
  - "Quarter" refers to fiscal quarter (Feb-Apr, May-Jul, Aug-Oct, Nov-Jan)
  - "Recent" means last 30 days unless specified
  - "Current" refers to current fiscal period
```

### Issue 16: Inconsistent Results

#### Symptoms
- Same question returns different results at different times
- Results change without data changes
- Inconsistency between similar queries

#### Diagnostic Process
```sql
-- Track result consistency
WITH query_results AS (
  SELECT 
    user_question,
    generated_sql,
    request_timestamp,
    ROW_NUMBER() OVER (PARTITION BY user_question ORDER BY request_timestamp) as query_sequence
  FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
    'SEMANTIC_VIEW', 
    'your_database.your_schema.your_semantic_view'
  ))
  WHERE user_question = 'What was our total revenue last month?'
    AND error_message IS NULL
    AND request_timestamp >= DATEADD('day', -30, CURRENT_TIMESTAMP())
)
SELECT 
  user_question,
  generated_sql,
  query_sequence,
  CASE 
    WHEN LAG(generated_sql) OVER (ORDER BY query_sequence) = generated_sql 
    THEN 'CONSISTENT'
    ELSE 'DIFFERENT'
  END as consistency_check
FROM query_results
ORDER BY query_sequence;
```

#### Solutions

**Solution 1: Improve Model Stability**
```yaml
# Add more specific verified queries
verified_queries:
  - question: "What was our total revenue last month?"
    verified_at: "2024-01-15T10:30:00Z"
  
  - question: "What was our total sales last month?"  # Synonym variation
    verified_at: "2024-01-15T10:30:00Z"
  
  - question: "How much revenue did we generate last month?"  # Rephrasing
    verified_at: "2024-01-15T10:30:00Z"
```

**Solution 2: Add Constraints via Custom Instructions**
```yaml
custom_instructions: |
  Query Consistency Rules:
  
  ## Date Handling
  - "Last month" always refers to the complete previous calendar month
  - Use DATE_TRUNC('month', CURRENT_DATE - INTERVAL '1 month') for start date
  - Use LAST_DAY(CURRENT_DATE - INTERVAL '1 month') for end date
  
  ## Aggregation Rules
  - "Total revenue" includes all completed orders, excluding cancelled orders
  - Always use SUM(order_total) for revenue calculations
  - Include tax in revenue calculations unless specifically excluded
  
  ## Filtering Rules
  - Default to completed/shipped orders unless specified otherwise
  - Exclude test orders (is_test = false)
  - Include refunds as negative revenue
```

---

## Data Quality and Accuracy Problems

### Issue 17: Incorrect Query Results

#### Symptoms
- Query results don't match expected business values
- Calculations appear wrong
- Missing data in results

#### Data Validation Process
```sql
-- 1. Validate underlying data quality
SELECT 
  COUNT(*) as total_records,
  COUNT(DISTINCT customer_id) as unique_customers,
  COUNT(CASE WHEN order_total IS NULL THEN 1 END) as null_amounts,
  COUNT(CASE WHEN order_total < 0 THEN 1 END) as negative_amounts,
  COUNT(CASE WHEN order_date > CURRENT_DATE THEN 1 END) as future_dates
FROM orders;

-- 2. Check relationship integrity
SELECT 
  'Orders without customers' as issue,
  COUNT(*) as count
FROM orders o
LEFT JOIN customers c USING (customer_id)
WHERE c.customer_id IS NULL

UNION ALL

SELECT 
  'Order items without orders' as issue,
  COUNT(*) as count
FROM order_items oi
LEFT JOIN orders o USING (order_id)
WHERE o.order_id IS NULL;

-- 3. Validate business logic
SELECT 
  order_id,
  order_total,
  SUM(oi.quantity * oi.unit_price) as calculated_total,
  ABS(order_total - calculated_total) as difference
FROM orders o
JOIN order_items oi USING (order_id)
GROUP BY order_id, order_total
HAVING ABS(difference) > 0.01  -- Find discrepancies
ORDER BY difference DESC;
```

#### Data Quality Fixes
```sql
-- Create clean data views for semantic model
CREATE OR REPLACE VIEW clean_orders AS
SELECT 
  order_id,
  customer_id,
  order_date,
  CASE 
    WHEN order_total < 0 THEN 0  -- Handle negative amounts
    WHEN order_total IS NULL THEN 0  -- Handle nulls
    ELSE order_total 
  END as order_total,
  order_status
FROM orders
WHERE order_date <= CURRENT_DATE  -- Exclude future dates
  AND customer_id IS NOT NULL      -- Exclude orphaned records
  AND is_test = false;             -- Exclude test data

-- Update semantic model to use clean view
```

### Issue 18: Missing or Incomplete Data

#### Symptoms
- Query results show unexpectedly low numbers
- Time series with gaps
- Missing categories or segments in results

#### Data Completeness Analysis
```sql
-- Check for data gaps by time period
WITH date_spine AS (
  SELECT DATE_TRUNC('day', DATEADD('day', seq4(), '2023-01-01')) as calendar_date
  FROM TABLE(GENERATOR(rowcount => 365))
  WHERE calendar_date <= CURRENT_DATE
),
daily_orders AS (
  SELECT 
    DATE_TRUNC('day', order_date) as order_date,
    COUNT(*) as order_count,
    SUM(order_total) as daily_revenue
  FROM orders
  WHERE order_date >= '2023-01-01'
  GROUP BY DATE_TRUNC('day', order_date)
)
SELECT 
  ds.calendar_date,
  COALESCE(do.order_count, 0) as order_count,
  COALESCE(do.daily_revenue, 0) as daily_revenue,
  CASE WHEN do.order_date IS NULL THEN 'MISSING DATA' ELSE 'HAS DATA' END as data_status
FROM date_spine ds
LEFT JOIN daily_orders do ON ds.calendar_date = do.order_date
WHERE ds.calendar_date >= '2023-01-01'
ORDER BY ds.calendar_date;
```

#### Solutions for Data Completeness
```sql
-- Create comprehensive date dimension
CREATE OR REPLACE TABLE date_dimension AS
SELECT 
  date_value,
  YEAR(date_value) as year,
  QUARTER(date_value) as quarter,
  MONTH(date_value) as month,
  DAY(date_value) as day,
  DAYOFWEEK(date_value) as day_of_week,
  CASE WHEN DAYOFWEEK(date_value) IN (1, 7) THEN 'Weekend' ELSE 'Weekday' END as day_type
FROM (
  SELECT DATEADD('day', ROW_NUMBER() OVER (ORDER BY 1) - 1, '2020-01-01') as date_value
  FROM TABLE(GENERATOR(rowcount => 2000))
)
WHERE date_value <= CURRENT_DATE + 365;

-- Use in semantic model to ensure complete time series
```

---

## Emergency Response Procedures

### Critical Issue Response Checklist

#### 1. System Outage (No queries working)
```markdown
□ Check Snowflake account status
□ Verify warehouse availability  
□ Confirm network connectivity
□ Check user permissions
□ Review recent model changes
□ Contact Snowflake support if needed
```

#### 2. Data Accuracy Crisis (Wrong results in production)
```markdown
□ Immediately verify underlying data sources
□ Check recent ETL job status
□ Compare results with known good queries
□ Review recent semantic model changes
□ Consider rolling back to previous model version
□ Communicate with stakeholders about potential issues
```

#### 3. Performance Degradation (Queries timing out)
```markdown
□ Check warehouse utilization
□ Review recent query complexity trends
□ Identify resource-intensive queries
□ Scale up warehouse temporarily
□ Implement query result limiting
□ Schedule optimization work
```

### Escalation Procedures

#### Internal Escalation
1. **Level 1**: Check monitoring dashboards and logs
2. **Level 2**: Review semantic model and data quality
3. **Level 3**: Engage database administrators
4. **Level 4**: Contact business stakeholders

#### External Escalation (Snowflake Support)
1. Gather system information (account ID, warehouse details)
2. Document error messages and timestamps
3. Provide semantic model configuration
4. Include monitoring data and query examples
5. Submit support ticket with priority level

---

*This troubleshooting guide is based on official Snowflake Cortex Analyst documentation and real-world implementation experience. For the most current troubleshooting information, refer to the [official Snowflake documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst) and support resources.*
