# Practical Examples for Cortex Analyst

This guide provides real-world, actionable examples for implementing Cortex Analyst features based on official Snowflake documentation.

## Table of Contents
1. [Basic Semantic Model Implementation](#basic-semantic-model-implementation)
2. [Verified Query Repository Examples](#verified-query-repository-examples)
3. [Cortex Search Integration Examples](#cortex-search-integration-examples)
4. [Monitoring and Logging Examples](#monitoring-and-logging-examples)
5. [Custom Instructions Examples](#custom-instructions-examples)
6. [API Integration Examples](#api-integration-examples)
7. [Complete Implementation Workflows](#complete-implementation-workflows)

---

## Basic Semantic Model Implementation

### Example 1: E-commerce Semantic Model

**Scenario**: Building a semantic model for an online retail business with customer orders, products, and inventory data.

#### Step 1: Table Structure Setup
```yaml
# semantic_model.yaml
name: ecommerce_analytics
description: "E-commerce analytics semantic model for sales, customers, and inventory analysis"

tables:
  - name: customers
    base_table:
      database: retail_db
      schema: public
      table: customers
    
    dimensions:
      - name: customer_id
        expr: customer_id
        description: "Unique identifier for each customer"
        unique_identifier: true
        
      - name: customer_name
        expr: first_name || ' ' || last_name
        description: "Full customer name combining first and last name"
        
      - name: customer_segment
        expr: customer_segment
        description: "Customer classification: Premium, Standard, or Basic based on annual spending"
        synonyms: ["customer type", "customer tier", "segment"]
        
      - name: registration_date
        expr: registration_date
        description: "Date when customer first registered on the platform"
        
      - name: email_domain
        expr: SPLIT_PART(email, '@', 2)
        description: "Email domain to identify business vs personal customers"

  - name: products
    base_table:
      database: retail_db
      schema: public
      table: products
    
    dimensions:
      - name: product_id
        expr: product_id
        description: "Unique product identifier"
        unique_identifier: true
        
      - name: product_name
        expr: product_name
        description: "Full product name as displayed to customers"
        synonyms: ["item name", "product title"]
        
      - name: category
        expr: category
        description: "Primary product category like Electronics, Clothing, Home & Garden"
        synonyms: ["product category", "department"]
        
      - name: subcategory
        expr: subcategory
        description: "More specific product classification within category"
        
      - name: brand
        expr: brand
        description: "Product manufacturer or brand name"
        synonyms: ["manufacturer", "make"]
    
    facts:
      - name: unit_price
        expr: unit_price
        description: "Current selling price per unit in USD"
        synonyms: ["price", "cost", "selling price"]
        
      - name: cost_of_goods
        expr: cost_of_goods_sold
        description: "Cost to acquire or manufacture the product"

  - name: orders
    base_table:
      database: retail_db
      schema: public
      table: orders
    
    dimensions:
      - name: order_id
        expr: order_id
        description: "Unique identifier for each order"
        unique_identifier: true
        
      - name: customer_id
        expr: customer_id
        description: "Links to customer who placed the order"
        
      - name: order_date
        expr: order_date
        description: "Date when the order was placed"
        
      - name: order_status
        expr: order_status
        description: "Current status: Pending, Shipped, Delivered, Cancelled"
        synonyms: ["status", "order state"]
        
      - name: shipping_method
        expr: shipping_method
        description: "Delivery method chosen: Standard, Express, Overnight"
    
    facts:
      - name: order_total
        expr: order_total
        description: "Total order value including taxes and shipping"
        synonyms: ["total amount", "order value", "revenue"]
        
      - name: tax_amount
        expr: tax_amount
        description: "Sales tax charged on the order"
        
      - name: shipping_cost
        expr: shipping_cost
        description: "Cost charged for shipping and handling"

relationships:
  - name: orders_to_customers
    left_table: orders
    right_table: customers
    join_columns:
      - left_column: customer_id
        right_column: customer_id
    description: "Each order belongs to one customer"

  - name: order_items_to_orders
    left_table: order_items
    right_table: orders
    join_columns:
      - left_column: order_id
        right_column: order_id
    description: "Order items belong to specific orders"

  - name: order_items_to_products
    left_table: order_items
    right_table: products
    join_columns:
      - left_column: product_id
        right_column: product_id
    description: "Order items reference specific products"
```

#### Step 2: Testing the Model
```sql
-- Test basic functionality with SQL queries
-- These should work through Cortex Analyst after model deployment

-- Test 1: Simple aggregation
"What was our total revenue last month?"

-- Test 2: Cross-table relationship
"Which customers bought the most expensive products?"

-- Test 3: Category analysis  
"Show me sales by product category"

-- Test 4: Time-based analysis
"Compare this quarter's performance to last quarter"
```

---

## Verified Query Repository Examples

Based on [official VQR documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst/verified-query-repository), here are practical examples of building and maintaining a VQR.

### Example 2: Building a Comprehensive VQR

#### Step 1: Core Business Questions
```yaml
# Add to your semantic model YAML
verified_queries:
  # Sales Performance Queries
  - question: "What was our total revenue last month?"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "Show me sales trends for the last 6 months"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "Which products generated the most revenue this quarter?"
    verified_at: "2024-01-15T10:30:00Z"
    
  # Customer Analysis Queries
  - question: "How many new customers did we acquire this month?"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "What's the average order value by customer segment?"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "Which customer segment has the highest lifetime value?"
    verified_at: "2024-01-15T10:30:00Z"
    
  # Product Performance Queries
  - question: "What are our top 10 selling products by quantity?"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "Which product categories have the best profit margins?"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "Show me inventory levels for fast-moving products"
    verified_at: "2024-01-15T10:30:00Z"
```

#### Step 2: Query Variations
```yaml
# Include multiple phrasings for the same business question
verified_queries:
  # Revenue variations
  - question: "What was our total revenue last month?"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "How much money did we make last month?"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "Show me last month's sales total"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "What were our earnings for the previous month?"
    verified_at: "2024-01-15T10:30:00Z"
    
  # Top products variations
  - question: "Which products sell the most?"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "What are our best-selling items?"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "Show me top performing products"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "Which items have the highest sales volume?"
    verified_at: "2024-01-15T10:30:00Z"
```

#### Step 3: VQR Maintenance Workflow
```sql
-- Query to identify potential VQR candidates from monitoring logs
SELECT 
  user_question,
  COUNT(*) as question_frequency,
  AVG(CASE WHEN error_message IS NULL THEN 1 ELSE 0 END) as success_rate,
  ARRAY_AGG(DISTINCT user_name) as users_asking
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW', 
  'retail_db.public.ecommerce_analytics'
))
WHERE request_timestamp >= DATEADD('day', -30, CURRENT_TIMESTAMP())
  AND error_message IS NULL  -- Only successful queries
GROUP BY user_question
HAVING question_frequency >= 3  -- Asked at least 3 times
  AND success_rate >= 0.9      -- High success rate
ORDER BY question_frequency DESC;
```

---

## Cortex Search Integration Examples

Based on [official Cortex Search documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst/cortex-analyst-search-integration), here are practical implementation examples.

### Example 3: Product Name Search Integration

#### Scenario: Customers searching for products using various terms

**Problem**: Users ask questions like "Show sales for iPhone" but product names in database are "Apple iPhone 14 Pro Max 128GB Space Black"

#### Step 1: Create Cortex Search Service
```sql
-- Create search service for product names
CREATE OR REPLACE CORTEX SEARCH SERVICE product_name_search
  ON product_name
  WAREHOUSE = xsmall
  TARGET_LAG = '1 hour'
  AS (
    SELECT DISTINCT 
      product_id,
      product_name,
      brand,
      category
    FROM retail_db.public.products
  );
```

#### Step 2: Integrate with Semantic Model
```yaml
# Add to semantic model
tables:
  - name: products
    base_table:
      database: retail_db
      schema: public
      table: products
    
    dimensions:
      - name: product_name
        expr: product_name
        description: "Product names including brand, model, and specifications"
        synonyms: ["item name", "product title", "SKU name"]
        cortex_search_service:
          service: product_name_search
          literal_column: product_name
          database: retail_db
          schema: public
```

#### Step 3: Test the Integration
```sql
-- These natural language queries should now work better:

-- User query: "Show me iPhone sales"
-- Should find: "Apple iPhone 14 Pro Max", "Apple iPhone 13", etc.

-- User query: "Samsung phone revenue"  
-- Should find: "Samsung Galaxy S23", "Samsung Galaxy Note", etc.

-- User query: "laptop sales by brand"
-- Should find: various laptop models across brands
```

### Example 4: Customer Company Search

#### Scenario: B2B business with many company name variations

#### Step 1: Create Company Search Service
```sql
-- Handle company name variations
CREATE OR REPLACE CORTEX SEARCH SERVICE company_search
  ON company_name
  WAREHOUSE = xsmall  
  TARGET_LAG = '4 hours'
  AS (
    SELECT DISTINCT 
      customer_id,
      company_name,
      UPPER(company_name) as company_name_upper,
      -- Include common variations
      REGEXP_REPLACE(company_name, ' Inc\\.?| LLC| Corporation| Corp\\.?', '', 'i') as company_base_name
    FROM retail_db.public.customers
    WHERE company_name IS NOT NULL
  );
```

#### Step 2: Semantic Model Integration
```yaml
dimensions:
  - name: company_name
    expr: company_name
    description: "Customer company name with common business entity suffixes"
    synonyms: ["business name", "organization", "company"]
    cortex_search_service:
      service: company_search
      literal_column: company_name
```

---

## Monitoring and Logging Examples

Based on [official monitoring documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst/admin-observability), here are practical monitoring implementations.

### Example 5: Daily Operations Dashboard

#### Step 1: Create Monitoring Views
```sql
-- Daily metrics view
CREATE OR REPLACE VIEW cortex_analyst_daily_dashboard AS
WITH daily_stats AS (
  SELECT 
    DATE_TRUNC('day', request_timestamp) as metric_date,
    COUNT(*) as total_queries,
    COUNT(DISTINCT user_name) as unique_users,
    COUNT(CASE WHEN error_message IS NULL THEN 1 END) as successful_queries,
    COUNT(CASE WHEN error_message IS NOT NULL THEN 1 END) as failed_queries,
    AVG(CASE WHEN error_message IS NULL THEN 1 ELSE 0 END) as success_rate
  FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
    'SEMANTIC_VIEW', 
    'retail_db.public.ecommerce_analytics'
  ))
  WHERE request_timestamp >= DATEADD('day', -30, CURRENT_TIMESTAMP())
  GROUP BY DATE_TRUNC('day', request_timestamp)
)
SELECT 
  metric_date,
  total_queries,
  unique_users,
  successful_queries,
  failed_queries,
  ROUND(success_rate * 100, 2) as success_rate_percent,
  -- Calculate growth metrics
  LAG(total_queries) OVER (ORDER BY metric_date) as prev_day_queries,
  ROUND(
    ((total_queries - LAG(total_queries) OVER (ORDER BY metric_date)) / 
     LAG(total_queries) OVER (ORDER BY metric_date)) * 100, 
    2
  ) as query_growth_percent
FROM daily_stats
ORDER BY metric_date DESC;
```

#### Step 2: Error Analysis View
```sql
-- Top errors analysis
CREATE OR REPLACE VIEW cortex_analyst_error_analysis AS
SELECT 
  error_message,
  COUNT(*) as error_frequency,
  COUNT(DISTINCT user_name) as affected_users,
  ARRAY_AGG(DISTINCT user_question) as sample_questions,
  MIN(request_timestamp) as first_occurrence,
  MAX(request_timestamp) as last_occurrence
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW', 
  'retail_db.public.ecommerce_analytics'
))
WHERE error_message IS NOT NULL
  AND request_timestamp >= DATEADD('day', -7, CURRENT_TIMESTAMP())
GROUP BY error_message
ORDER BY error_frequency DESC;
```

#### Step 3: User Adoption Tracking
```sql
-- User engagement metrics
CREATE OR REPLACE VIEW cortex_analyst_user_adoption AS
WITH user_stats AS (
  SELECT 
    user_name,
    COUNT(*) as total_queries,
    COUNT(DISTINCT DATE_TRUNC('day', request_timestamp)) as active_days,
    MIN(request_timestamp) as first_query_date,
    MAX(request_timestamp) as last_query_date,
    AVG(CASE WHEN error_message IS NULL THEN 1 ELSE 0 END) as personal_success_rate
  FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
    'SEMANTIC_VIEW', 
    'retail_db.public.ecommerce_analytics'
  ))
  WHERE request_timestamp >= DATEADD('day', -30, CURRENT_TIMESTAMP())
  GROUP BY user_name
)
SELECT 
  user_name,
  total_queries,
  active_days,
  ROUND(total_queries / active_days, 2) as avg_queries_per_day,
  DATEDIFF('day', first_query_date, last_query_date) as usage_span_days,
  ROUND(personal_success_rate * 100, 2) as success_rate_percent,
  CASE 
    WHEN active_days >= 20 THEN 'Heavy User'
    WHEN active_days >= 10 THEN 'Regular User'
    WHEN active_days >= 3 THEN 'Occasional User'
    ELSE 'New User'
  END as user_category
FROM user_stats
ORDER BY total_queries DESC;
```

### Example 6: Automated Alerts

#### Step 1: Success Rate Alert
```sql
-- Create task for daily success rate monitoring
CREATE OR REPLACE TASK monitor_cortex_analyst_success_rate
  WAREHOUSE = 'CORTEX_MONITORING_WH'
  SCHEDULE = 'USING CRON 0 9 * * * UTC'  -- Daily at 9 AM UTC
AS
DECLARE
  success_rate_threshold NUMBER DEFAULT 80.0;
  yesterday_success_rate NUMBER;
BEGIN
  -- Calculate yesterday's success rate
  SELECT 
    AVG(CASE WHEN error_message IS NULL THEN 1 ELSE 0 END) * 100
  INTO yesterday_success_rate
  FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
    'SEMANTIC_VIEW', 
    'retail_db.public.ecommerce_analytics'
  ))
  WHERE DATE_TRUNC('day', request_timestamp) = CURRENT_DATE - 1;
  
  -- Send alert if below threshold
  IF (yesterday_success_rate < success_rate_threshold) THEN
    -- Insert alert into monitoring table or send notification
    INSERT INTO cortex_analyst_alerts (
      alert_date,
      alert_type,
      metric_value,
      threshold_value,
      alert_message
    ) VALUES (
      CURRENT_DATE - 1,
      'LOW_SUCCESS_RATE',
      yesterday_success_rate,
      success_rate_threshold,
      'Cortex Analyst success rate dropped to ' || yesterday_success_rate || '%'
    );
  END IF;
END;

-- Start the task
ALTER TASK monitor_cortex_analyst_success_rate RESUME;
```

---

## Custom Instructions Examples

### Example 7: Business-Specific Instructions

#### Retail Business Custom Instructions
```yaml
custom_instructions: |
  Business Context for E-commerce Analytics:
  
  ## Date and Time Handling
  - Business operates in EST timezone
  - Fiscal year runs from February 1 to January 31
  - "Current quarter" refers to fiscal quarter (Q1: Feb-Apr, Q2: May-Jul, Q3: Aug-Oct, Q4: Nov-Jan)
  - "Recent" or "latest" refers to the last 30 days unless specified
  - Weekend sales patterns differ significantly - consider separate analysis
  
  ## Financial Calculations
  - All revenue amounts should be in USD
  - Include tax in total revenue calculations
  - Exclude shipping costs from product revenue analysis
  - Round monetary values to 2 decimal places
  
  ## Customer Segmentation
  - Premium customers: Annual spend > $5,000
  - Standard customers: Annual spend $1,000 - $5,000  
  - Basic customers: Annual spend < $1,000
  - New customers: Registered within last 90 days
  
  ## Product Analysis
  - Focus on shipped orders only for sales metrics
  - Include cancelled orders in conversion analysis
  - Seasonal products (holiday, summer) need year-over-year comparison
  - Bundle products should be analyzed both as bundles and individual items
  
  ## Reporting Preferences
  - Always show percentage changes when comparing periods
  - Include sample size when showing averages
  - For top/bottom lists, default to top 10 unless specified
  - When showing trends, include confidence indicators for seasonal data
```

#### Healthcare Custom Instructions
```yaml
custom_instructions: |
  Healthcare Analytics Guidelines:
  
  ## Privacy and Compliance
  - Never display individual patient data
  - Always aggregate patient information (minimum 10 patients)
  - Use age groups instead of specific ages: 0-17, 18-35, 36-55, 56-75, 75+
  - Mask any potentially identifying information
  
  ## Clinical Context
  - Primary diagnosis takes precedence over secondary diagnoses
  - Include only completed visits for outcome analysis
  - Emergency visits vs scheduled visits require separate analysis
  - Consider readmission within 30 days as quality metric
  
  ## Time Periods
  - Clinical reporting follows calendar year
  - "Recent" refers to last 90 days for patient patterns
  - Seasonal illness patterns: include year-over-year comparisons
  - Weekend vs weekday patterns important for staffing analysis
  
  ## Quality Metrics
  - Length of stay calculations exclude same-day visits
  - Patient satisfaction scores: include response rate context
  - Provider performance: minimum 20 cases for statistical relevance
  - Cost analysis: include both direct and allocated costs
```

### Example 8: Technical Preferences

```yaml
custom_instructions: |
  Technical SQL Generation Preferences:
  
  ## Join Strategy
  - Use LEFT JOIN for optional relationships to preserve all records
  - Prefer explicit joins over WHERE clause joins
  - Include proper NULL handling in all aggregations
  
  ## Performance Optimization
  - Use LIMIT clauses for large result sets
  - Prefer window functions over correlated subqueries
  - Use CTEs for complex multi-step calculations
  - Include appropriate indexes hints when beneficial
  
  ## Data Quality
  - Always filter out test/demo data (where is_test = false)
  - Exclude soft-deleted records (where deleted_at IS NULL)
  - Handle timezone conversions explicitly
  - Validate date ranges for reasonableness
  
  ## Formatting Standards
  - Round percentages to 1 decimal place
  - Round monetary values to 2 decimal places  
  - Use consistent date formats: YYYY-MM-DD
  - Format large numbers with thousands separators
```

---

## API Integration Examples

### Example 9: REST API Usage

#### Basic API Call Example
```python
import requests
import json

def query_cortex_analyst(question, semantic_model):
    """
    Query Cortex Analyst using REST API
    """
    # Snowflake account details
    account_url = "https://your-account.snowflakecomputing.com"
    api_endpoint = f"{account_url}/api/v2/cortex/analyst/message"
    
    # Authentication headers
    headers = {
        "Authorization": f"Bearer {get_snowflake_token()}",
        "Content-Type": "application/json",
        "Accept": "application/json"
    }
    
    # Request payload
    payload = {
        "messages": [
            {
                "role": "user", 
                "content": [
                    {
                        "type": "text",
                        "text": question
                    }
                ]
            }
        ],
        "semantic_model": semantic_model
    }
    
    try:
        response = requests.post(api_endpoint, headers=headers, json=payload)
        response.raise_for_status()
        return response.json()
    
    except requests.exceptions.RequestException as e:
        print(f"API request failed: {e}")
        return None

# Example usage
def main():
    semantic_model = {
        "type": "semantic_view",
        "name": "retail_db.public.ecommerce_analytics"
    }
    
    questions = [
        "What was our total revenue last month?",
        "Which products are our top sellers?",
        "Show me customer acquisition trends"
    ]
    
    for question in questions:
        print(f"Question: {question}")
        result = query_cortex_analyst(question, semantic_model)
        
        if result:
            # Extract SQL and results
            sql_query = result.get('message', {}).get('content', [{}])[0].get('sql')
            print(f"Generated SQL: {sql_query}")
            print("---")
        else:
            print("Failed to get response")
            print("---")

if __name__ == "__main__":
    main()
```

#### Advanced API Integration with Error Handling
```python
import requests
import json
import time
from typing import Dict, List, Optional

class CortexAnalystClient:
    def __init__(self, account_url: str, token: str):
        self.account_url = account_url
        self.token = token
        self.base_url = f"{account_url}/api/v2/cortex/analyst"
        
    def _make_request(self, endpoint: str, payload: Dict) -> Optional[Dict]:
        """Make API request with retry logic"""
        headers = {
            "Authorization": f"Bearer {self.token}",
            "Content-Type": "application/json",
            "Accept": "application/json"
        }
        
        for attempt in range(3):  # Retry up to 3 times
            try:
                response = requests.post(
                    f"{self.base_url}/{endpoint}",
                    headers=headers,
                    json=payload,
                    timeout=30
                )
                
                if response.status_code == 200:
                    return response.json()
                elif response.status_code == 429:  # Rate limited
                    print(f"Rate limited, waiting {2**attempt} seconds...")
                    time.sleep(2**attempt)
                    continue
                else:
                    print(f"API error {response.status_code}: {response.text}")
                    return None
                    
            except requests.exceptions.RequestException as e:
                print(f"Request failed (attempt {attempt + 1}): {e}")
                if attempt < 2:
                    time.sleep(2**attempt)
                    
        return None
    
    def query(self, question: str, semantic_model: Dict) -> Optional[Dict]:
        """Send question to Cortex Analyst"""
        payload = {
            "messages": [
                {
                    "role": "user",
                    "content": [{"type": "text", "text": question}]
                }
            ],
            "semantic_model": semantic_model
        }
        
        return self._make_request("message", payload)
    
    def batch_query(self, questions: List[str], semantic_model: Dict) -> List[Dict]:
        """Process multiple questions"""
        results = []
        
        for i, question in enumerate(questions):
            print(f"Processing question {i+1}/{len(questions)}: {question}")
            
            result = self.query(question, semantic_model)
            results.append({
                "question": question,
                "result": result,
                "success": result is not None
            })
            
            # Add delay between requests to avoid rate limiting
            time.sleep(1)
            
        return results

# Usage example
def demo_batch_analysis():
    client = CortexAnalystClient(
        account_url="https://your-account.snowflakecomputing.com",
        token="your-token-here"
    )
    
    semantic_model = {
        "type": "semantic_view", 
        "name": "retail_db.public.ecommerce_analytics"
    }
    
    business_questions = [
        "What's our revenue trend over the last 6 months?",
        "Which customer segments are growing fastest?", 
        "What products have declining sales?",
        "How does customer lifetime value vary by acquisition channel?",
        "What's our average order value by region?"
    ]
    
    results = client.batch_query(business_questions, semantic_model)
    
    # Process results
    successful_queries = [r for r in results if r["success"]]
    print(f"Successfully processed {len(successful_queries)}/{len(results)} queries")
    
    return results
```

---

## Complete Implementation Workflows

### Example 10: End-to-End Implementation

#### Phase 1: Environment Setup
```sql
-- 1. Create database and schema
CREATE DATABASE IF NOT EXISTS cortex_analytics;
CREATE SCHEMA IF NOT EXISTS cortex_analytics.semantic_models;

-- 2. Create warehouse for Cortex Analyst
CREATE WAREHOUSE IF NOT EXISTS cortex_analyst_wh 
  WITH WAREHOUSE_SIZE = 'XSMALL'
       AUTO_SUSPEND = 300
       AUTO_RESUME = TRUE
       COMMENT = 'Warehouse for Cortex Analyst queries';

-- 3. Set up permissions
GRANT USAGE ON DATABASE cortex_analytics TO ROLE analyst_role;
GRANT USAGE ON SCHEMA cortex_analytics.semantic_models TO ROLE analyst_role;
GRANT USAGE ON WAREHOUSE cortex_analyst_wh TO ROLE analyst_role;
```

#### Phase 2: Data Preparation
```sql
-- 1. Create or identify source tables
-- Ensure proper data types and constraints

-- 2. Create summary/mart tables for performance
CREATE OR REPLACE TABLE cortex_analytics.semantic_models.customer_metrics AS
SELECT 
  customer_id,
  customer_name,
  customer_segment,
  registration_date,
  COUNT(DISTINCT order_id) as lifetime_orders,
  SUM(order_total) as lifetime_revenue,
  AVG(order_total) as avg_order_value,
  MAX(order_date) as last_order_date,
  DATEDIFF('day', MAX(order_date), CURRENT_DATE) as days_since_last_order
FROM retail_db.public.orders o
JOIN retail_db.public.customers c USING (customer_id)
GROUP BY customer_id, customer_name, customer_segment, registration_date;

-- 3. Create indexes for performance
CREATE INDEX IF NOT EXISTS idx_customer_metrics_segment 
  ON cortex_analytics.semantic_models.customer_metrics (customer_segment);
```

#### Phase 3: Semantic Model Development
```yaml
# Complete semantic model with all features
name: comprehensive_ecommerce_analytics
description: "Complete e-commerce analytics with customer, product, and sales analysis"

# Custom instructions for business context
custom_instructions: |
  E-commerce Business Rules:
  - Fiscal year: February to January
  - Customer segments: Premium (>$5K), Standard ($1K-5K), Basic (<$1K)
  - Include shipping in total revenue
  - Weekend patterns differ from weekdays
  - Focus on shipped orders for sales metrics

tables:
  - name: customer_metrics
    base_table:
      database: cortex_analytics
      schema: semantic_models
      table: customer_metrics
    
    dimensions:
      - name: customer_id
        expr: customer_id
        unique_identifier: true
        description: "Unique customer identifier"
      
      - name: customer_segment
        expr: customer_segment
        description: "Customer value tier: Premium, Standard, or Basic"
        synonyms: ["customer tier", "customer type", "segment"]
    
    facts:
      - name: lifetime_revenue
        expr: lifetime_revenue
        description: "Total revenue from customer across all orders"
        synonyms: ["customer value", "total spent", "CLV"]
      
      - name: avg_order_value
        expr: avg_order_value
        description: "Average amount spent per order"
        synonyms: ["AOV", "average order size"]

# Verified queries from business stakeholders
verified_queries:
  - question: "What's our total revenue this quarter?"
    verified_at: "2024-01-15T10:30:00Z"
  
  - question: "Which customer segment generates the most revenue?"
    verified_at: "2024-01-15T10:30:00Z"
  
  - question: "How many premium customers do we have?"
    verified_at: "2024-01-15T10:30:00Z"
  
  - question: "What's the average order value by customer segment?"
    verified_at: "2024-01-15T10:30:00Z"
```

#### Phase 4: Testing and Validation
```sql
-- Test queries to validate semantic model
-- Run these through Cortex Analyst interface

-- Basic functionality tests
"What was our revenue last month?"
"How many customers do we have?"
"Show me top 10 products by sales"

-- Relationship tests  
"Which customers buy premium products?"
"Show revenue by customer segment"
"What's the average order value for premium customers?"

-- Complex analysis tests
"Compare this quarter's performance to last quarter"
"Show customer acquisition trends by month"
"Which products have declining sales?"

-- Edge case tests
"Show data for customers with no orders"  -- Should handle gracefully
"What's revenue for invalid date range?"   -- Should provide helpful error
"Show products that don't exist"           -- Should return empty result
```

#### Phase 5: Monitoring Setup
```sql
-- Create monitoring dashboard tables
CREATE OR REPLACE TABLE cortex_analytics.monitoring.daily_metrics (
  metric_date DATE,
  total_queries INTEGER,
  unique_users INTEGER,
  success_rate FLOAT,
  avg_response_time FLOAT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP()
);

-- Create daily monitoring task
CREATE OR REPLACE TASK cortex_analytics.monitoring.daily_metrics_task
  WAREHOUSE = cortex_analyst_wh
  SCHEDULE = 'USING CRON 0 1 * * * UTC'  -- Daily at 1 AM UTC
AS
INSERT INTO cortex_analytics.monitoring.daily_metrics (
  metric_date,
  total_queries,
  unique_users, 
  success_rate
)
SELECT 
  CURRENT_DATE - 1 as metric_date,
  COUNT(*) as total_queries,
  COUNT(DISTINCT user_name) as unique_users,
  AVG(CASE WHEN error_message IS NULL THEN 1 ELSE 0 END) as success_rate
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW',
  'cortex_analytics.semantic_models.comprehensive_ecommerce_analytics'
))
WHERE DATE_TRUNC('day', request_timestamp) = CURRENT_DATE - 1;

-- Start monitoring
ALTER TASK cortex_analytics.monitoring.daily_metrics_task RESUME;
```

#### Phase 6: User Rollout Strategy
```markdown
# User Rollout Plan

## Week 1: Pilot Group (5 users)
- Data analysts and BI team members
- Intensive testing and feedback collection
- Daily check-ins and issue resolution

## Week 2: Department Rollout (20 users)
- Sales and marketing teams
- Provide training sessions
- Document common questions and issues

## Week 3: Company-wide Rollout (100+ users)
- All business users
- Self-service training materials
- Office hours for support

## Ongoing: Optimization
- Weekly VQR updates based on popular queries
- Monthly model refinements
- Quarterly feature enhancements
```

---

*These practical examples are based on official Snowflake Cortex Analyst documentation and real-world implementation patterns. For the most current API specifications and features, refer to the [official Snowflake documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst).*
