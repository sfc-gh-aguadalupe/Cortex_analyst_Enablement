# Cortex Analyst Best Practices

This guide provides official recommendations and proven strategies for implementing Cortex Analyst effectively in your organization.

## Table of Contents
1. [Semantic Model Design Best Practices](#semantic-model-design-best-practices)
2. [Verified Query Repository Best Practices](#verified-query-repository-best-practices)
3. [Cortex Search Integration Best Practices](#cortex-search-integration-best-practices)
4. [Monitoring and Observability Best Practices](#monitoring-and-observability-best-practices)
5. [Custom Instructions Best Practices](#custom-instructions-best-practices)
6. [Team Collaboration and Governance](#team-collaboration-and-governance)
7. [Performance Optimization](#performance-optimization)

---

## Semantic Model Design Best Practices

### Model Structure and Organization

#### 1. Clear Naming Conventions
```yaml
# ✅ Good - Descriptive and consistent
tables:
  - name: customer_orders
    dimensions:
      - name: customer_id
      - name: order_date
    facts:
      - name: order_amount
      - name: quantity_ordered

# ❌ Avoid - Unclear abbreviations
tables:
  - name: co
    dimensions:
      - name: cid
      - name: od
```

#### 2. Proper Dimension vs Fact Classification
```yaml
# ✅ Correct classification
dimensions:
  - name: customer_id          # Identifier
  - name: product_category     # Descriptive attribute
  - name: order_date          # Time dimension
  
facts:
  - name: revenue             # Measurable numeric value
  - name: quantity_sold       # Countable measure
  - name: profit_margin       # Calculated metric
```

#### 3. Comprehensive Business Descriptions
```yaml
# ✅ Excellent descriptions with business context
dimensions:
  - name: customer_segment
    description: "Customer classification based on purchase behavior and value. Segments include Premium (>$10K annual spend), Standard ($1K-$10K), and Basic (<$1K)."
    synonyms: ["customer type", "client category", "customer tier"]

facts:
  - name: monthly_recurring_revenue
    description: "Predictable revenue generated from active subscriptions each month, excluding one-time fees and variable usage charges."
    synonyms: ["MRR", "recurring revenue", "subscription revenue"]
```

### Relationship Design

#### 1. Start with Core Business Relationships
```yaml
# ✅ Implement fundamental business relationships first
relationships:
  - name: orders_to_customers
    left_table: orders
    right_table: customers
    join_columns:
      - left_column: customer_id
        right_column: customer_id
    
  - name: order_items_to_products
    left_table: order_items
    right_table: products
    join_columns:
      - left_column: product_id
        right_column: product_id
```

#### 2. Use Descriptive Relationship Names
```yaml
# ✅ Clear relationship purposes
relationships:
  - name: sales_to_sales_representatives
  - name: products_to_product_categories
  - name: customers_to_geographic_regions

# ❌ Avoid generic names
relationships:
  - name: table1_to_table2
  - name: join1
```

---

## Verified Query Repository Best Practices

Based on [official Snowflake VQR documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst/verified-query-repository), the Verified Query Repository improves model accuracy by storing validated question-answer pairs.

### Building an Effective VQR

#### 1. Start with Core Business Questions
```yaml
# ✅ Focus on frequently asked business questions
verified_queries:
  - question: "What was our total revenue last quarter?"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "Which products had the highest sales volume this month?"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "Show me customer acquisition trends by region"
    verified_at: "2024-01-15T10:30:00Z"
```

#### 2. Include Question Variations
```yaml
# ✅ Multiple ways users might ask the same question
verified_queries:
  - question: "What are our top selling products?"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "Which items sell the most?"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "Show me best performing products by sales volume"
    verified_at: "2024-01-15T10:30:00Z"
```

#### 3. Regular VQR Maintenance
- **Weekly Reviews**: Validate new queries that performed well
- **Monthly Audits**: Remove outdated or incorrect verified queries
- **Quarterly Updates**: Refresh VQR based on changing business needs
- **User Feedback Integration**: Add commonly requested queries to VQR

### VQR Implementation Strategy

#### Phase 1: Foundation (Weeks 1-2)
- Identify 10-15 core business questions
- Test and verify accuracy manually
- Add to VQR with proper timestamps

#### Phase 2: Expansion (Weeks 3-4)
- Analyze user query patterns from monitoring logs
- Add variations of successful queries
- Include edge cases and complex scenarios

#### Phase 3: Optimization (Ongoing)
- Regular performance review using monitoring data
- User feedback incorporation
- Continuous refinement based on usage patterns

---

## Cortex Search Integration Best Practices

Based on [official Cortex Search integration documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst/cortex-analyst-search-integration), Cortex Search improves literal string matching for more accurate SQL generation.

### When to Implement Cortex Search

#### 1. High-Cardinality Text Dimensions
```yaml
# ✅ Ideal candidates for Cortex Search
dimensions:
  - name: product_name          # Thousands of unique product names
    cortex_search_service:
      service: product_search_service
      
  - name: customer_company      # Many unique company names
    cortex_search_service:
      service: company_search_service
      
  - name: location_description  # Various location formats
    cortex_search_service:
      service: location_search_service
```

#### 2. Text Fields with Variations
```yaml
# ✅ Fields where users might use different terminology
dimensions:
  - name: product_category
    description: "Product categories including variations like 'Electronics', 'Tech', 'Technology'"
    cortex_search_service:
      service: category_search_service
```

### Cortex Search Service Setup

#### Option 1: UI-Based Setup (Recommended for Beginners)
1. Use the Cortex Analyst semantic model creation UI in Snowsight
2. Navigate to AI & ML » Cortex Analyst » Create new model
3. During dimension setup, select high-cardinality text columns
4. The UI automatically creates and links Cortex Search services

#### Option 2: Manual SQL Setup (Advanced Users)
```sql
-- Create Cortex Search Service for product names
CREATE OR REPLACE CORTEX SEARCH SERVICE product_search_service
  ON product_name
  WAREHOUSE = xsmall
  TARGET_LAG = '1 hour'
  AS (
    SELECT DISTINCT product_name 
    FROM products
  );
```

```yaml
# Link to semantic model
dimensions:
  - name: product_name
    expr: product_name
    cortex_search_service:
      service: product_search_service
      literal_column: product_name
      database: my_database
      schema: my_schema
```

### Cortex Search Performance Optimization

#### 1. Appropriate Warehouse Sizing
```sql
-- ✅ Right-size warehouse for your data volume
CREATE CORTEX SEARCH SERVICE my_search_service
  ON my_dimension
  WAREHOUSE = xsmall    -- For < 1M records
  -- WAREHOUSE = small  -- For 1M-10M records
  -- WAREHOUSE = medium -- For 10M+ records
  TARGET_LAG = '1 hour';
```

#### 2. Optimal Target Lag Settings
```sql
-- ✅ Balance freshness with cost
TARGET_LAG = '1 hour'    -- For frequently changing data
-- TARGET_LAG = '4 hours' -- For daily updated data
-- TARGET_LAG = '24 hours' -- For static reference data
```

---

## Monitoring and Observability Best Practices

Based on [official administrator monitoring documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst/admin-observability), effective monitoring is crucial for maintaining and improving Cortex Analyst performance.

### Essential Monitoring Queries

#### 1. Query Success Rate Monitoring
```sql
-- Monitor overall query success rate
SELECT 
  DATE_TRUNC('day', request_timestamp) as query_date,
  COUNT(*) as total_queries,
  SUM(CASE WHEN error_message IS NULL THEN 1 ELSE 0 END) as successful_queries,
  (successful_queries / total_queries) * 100 as success_rate_percent
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW', 
  'your_database.your_schema.your_semantic_view'
))
WHERE request_timestamp >= DATEADD('day', -7, CURRENT_TIMESTAMP())
GROUP BY query_date
ORDER BY query_date DESC;
```

#### 2. User Adoption Tracking
```sql
-- Track user engagement and adoption
SELECT 
  user_name,
  COUNT(*) as total_queries,
  COUNT(DISTINCT DATE_TRUNC('day', request_timestamp)) as active_days,
  AVG(CASE WHEN error_message IS NULL THEN 1 ELSE 0 END) as success_rate
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW', 
  'your_database.your_schema.your_semantic_view'
))
WHERE request_timestamp >= DATEADD('day', -30, CURRENT_TIMESTAMP())
GROUP BY user_name
HAVING total_queries >= 5
ORDER BY total_queries DESC;
```

#### 3. Common Error Analysis
```sql
-- Identify and analyze common errors
SELECT 
  error_message,
  COUNT(*) as error_count,
  ARRAY_AGG(DISTINCT user_question) as sample_questions
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW', 
  'your_database.your_schema.your_semantic_view'
))
WHERE error_message IS NOT NULL
  AND request_timestamp >= DATEADD('day', -7, CURRENT_TIMESTAMP())
GROUP BY error_message
ORDER BY error_count DESC
LIMIT 10;
```

### Monitoring Dashboard Setup

#### 1. Daily Health Check Dashboard
Create views for common monitoring metrics:

```sql
-- Create monitoring view for daily dashboard
CREATE OR REPLACE VIEW cortex_analyst_daily_metrics AS
SELECT 
  DATE_TRUNC('day', request_timestamp) as metric_date,
  COUNT(*) as total_queries,
  COUNT(DISTINCT user_name) as unique_users,
  AVG(CASE WHEN error_message IS NULL THEN 1 ELSE 0 END) as success_rate,
  COUNT(CASE WHEN error_message IS NOT NULL THEN 1 END) as error_count
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW', 
  'your_database.your_schema.your_semantic_view'
))
WHERE request_timestamp >= DATEADD('day', -30, CURRENT_TIMESTAMP())
GROUP BY metric_date;
```

#### 2. Performance Monitoring
```sql
-- Monitor query performance and complexity
SELECT 
  LENGTH(generated_sql) as sql_complexity,
  user_question,
  request_timestamp,
  CASE 
    WHEN error_message IS NULL THEN 'Success'
    ELSE 'Error'
  END as query_status
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW', 
  'your_database.your_schema.your_semantic_view'
))
WHERE request_timestamp >= DATEADD('day', -1, CURRENT_TIMESTAMP())
ORDER BY sql_complexity DESC;
```

### Monitoring Alert Setup

#### 1. Success Rate Alerts
```sql
-- Create alert for low success rates
-- (Implement as scheduled task or external monitoring)
SELECT 
  'ALERT: Low success rate' as alert_type,
  success_rate_percent,
  query_date
FROM (
  SELECT 
    DATE_TRUNC('day', request_timestamp) as query_date,
    (COUNT(CASE WHEN error_message IS NULL THEN 1 END) / COUNT(*)) * 100 as success_rate_percent
  FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
    'SEMANTIC_VIEW', 
    'your_database.your_schema.your_semantic_view'
  ))
  WHERE request_timestamp >= DATEADD('day', -1, CURRENT_TIMESTAMP())
  GROUP BY query_date
)
WHERE success_rate_percent < 80;  -- Alert threshold
```

---

## Custom Instructions Best Practices

Custom instructions help fine-tune Cortex Analyst behavior for specific business contexts and requirements.

### Effective Custom Instructions

#### 1. Business Context Instructions
```yaml
custom_instructions: |
  When analyzing sales data:
  - Always consider seasonal patterns in retail data
  - Fiscal year starts in February for this company
  - Include currency conversion when dealing with international sales
  - Default time period for "recent" is last 30 days unless specified
```

#### 2. Domain-Specific Guidelines
```yaml
# Healthcare example
custom_instructions: |
  For patient analytics:
  - Always aggregate data to protect patient privacy
  - Use age groups instead of specific ages
  - Consider only completed visits for outcome analysis
  - Default to current year for trend analysis
```

#### 3. Technical Preferences
```yaml
custom_instructions: |
  Query preferences:
  - Always use LEFT JOIN for optional relationships
  - Include null handling in aggregations
  - Prefer window functions over subqueries when possible
  - Round financial amounts to 2 decimal places
```

### Custom Instructions Implementation

#### 1. Start Simple and Iterate
```yaml
# ✅ Begin with essential business rules
custom_instructions: |
  - Use fiscal year (Feb-Jan) for all yearly calculations
  - Default currency is USD
  - Round monetary values to 2 decimal places

# 🔄 Evolve based on user feedback
custom_instructions: |
  - Use fiscal year (Feb-Jan) for all yearly calculations
  - Default currency is USD, convert international sales using latest rates
  - Round monetary values to 2 decimal places
  - When showing trends, include year-over-year comparison
  - For customer analysis, segment by value: Premium (>$10K), Standard ($1K-$10K), Basic (<$1K)
```

#### 2. Document Instructions Clearly
```yaml
custom_instructions: |
  ## Date Handling
  - Fiscal year: February 1 to January 31
  - "Current quarter" refers to fiscal quarter
  - "Last year" means same period in previous fiscal year
  
  ## Financial Calculations
  - All revenue in USD (convert if needed)
  - Round to 2 decimal places
  - Include tax in revenue calculations
  
  ## Customer Segmentation
  - Premium: Annual spend > $10,000
  - Standard: Annual spend $1,000 - $10,000
  - Basic: Annual spend < $1,000
```

---

## Team Collaboration and Governance

### Model Development Workflow

#### 1. Development Lifecycle
```mermaid
Development → Testing → Review → Deployment → Monitoring → Iteration
```

**Development Phase:**
- Create semantic models in development environment
- Test with sample data and known queries
- Document all design decisions

**Testing Phase:**
- Validate against business requirements
- Test edge cases and error conditions
- Performance testing with production data volumes

**Review Phase:**
- Business stakeholder validation
- Technical peer review
- Security and compliance review

**Deployment Phase:**
- Staged deployment (dev → test → prod)
- Rollback plan preparation
- User communication and training

**Monitoring Phase:**
- Track usage and performance metrics
- Monitor error rates and user feedback
- Regular model health checks

**Iteration Phase:**
- Incorporate user feedback
- Add new verified queries
- Optimize based on monitoring insights

#### 2. Role-Based Responsibilities

**Data Engineers:**
- Semantic model creation and maintenance
- Technical implementation of integrations
- Performance optimization
- Data pipeline management

**Business Analysts:**
- Business requirements definition
- Verified query creation and validation
- User training and support
- Business logic validation

**Data Governance Team:**
- Access control and security policies
- Data quality standards
- Compliance and audit requirements
- Model approval workflows

**End Users:**
- Feedback on model accuracy and usability
- New business question identification
- Adoption and usage

### Version Control and Change Management

#### 1. Semantic Model Versioning
```yaml
# Include version metadata in models
metadata:
  version: "2.1.0"
  created_date: "2024-01-15"
  last_modified: "2024-03-01"
  authors: ["data-team@company.com"]
  changelog:
    - "2.1.0: Added customer lifetime value calculations"
    - "2.0.0: Major restructure for new product lines"
    - "1.5.0: Added Cortex Search integration"
```

#### 2. Change Documentation
```markdown
# Change Log Template

## Version 2.1.0 - 2024-03-01

### Added
- Customer lifetime value calculation
- New product category dimensions
- Verified queries for customer analysis

### Modified
- Updated customer segmentation logic
- Improved product hierarchy relationships

### Deprecated
- Legacy customer classification method

### Removed
- Outdated promotional category dimensions

### Business Impact
- Enables new customer value analysis
- Improves product performance reporting
- Supports new pricing strategy initiatives
```

---

## Performance Optimization

### Query Performance Best Practices

#### 1. Efficient Semantic Model Design
```yaml
# ✅ Optimize for common query patterns
tables:
  - name: sales_summary
    base_table:
      # Use pre-aggregated tables for common metrics
      database: analytics_db
      schema: marts
      table: daily_sales_summary
    
    facts:
      - name: daily_revenue
      - name: transaction_count
      - name: average_order_value
```

#### 2. Strategic Use of Materialized Views
```sql
-- Create materialized views for frequently accessed aggregations
CREATE MATERIALIZED VIEW customer_monthly_metrics AS
SELECT 
  customer_id,
  DATE_TRUNC('month', order_date) as order_month,
  SUM(order_amount) as monthly_revenue,
  COUNT(*) as monthly_orders,
  AVG(order_amount) as avg_order_value
FROM orders
GROUP BY customer_id, DATE_TRUNC('month', order_date);
```

#### 3. Appropriate Warehouse Sizing
```sql
-- Size warehouses based on typical query complexity
-- XS: Simple aggregations, small datasets
-- S:  Complex joins, medium datasets  
-- M:  Advanced analytics, large datasets
-- L+: Heavy computation, very large datasets

-- Example warehouse assignment strategy
USE WAREHOUSE cortex_analyst_xs;  -- For basic metrics
USE WAREHOUSE cortex_analyst_m;   -- For complex analysis
```

### Data Freshness Optimization

#### 1. Incremental Data Loading
```sql
-- Implement incremental updates for better performance
MERGE INTO customer_metrics cm
USING (
  SELECT 
    customer_id,
    SUM(order_amount) as total_revenue,
    COUNT(*) as total_orders
  FROM orders 
  WHERE order_date >= CURRENT_DATE - 1  -- Only process recent data
  GROUP BY customer_id
) src ON cm.customer_id = src.customer_id
WHEN MATCHED THEN UPDATE SET
  total_revenue = src.total_revenue,
  total_orders = src.total_orders,
  last_updated = CURRENT_TIMESTAMP()
WHEN NOT MATCHED THEN INSERT VALUES
  (src.customer_id, src.total_revenue, src.total_orders, CURRENT_TIMESTAMP());
```

#### 2. Smart Refresh Strategies
```yaml
# Configure refresh frequency based on data volatility
tables:
  - name: real_time_sales      # High frequency updates
    refresh_frequency: "5 minutes"
    
  - name: customer_analytics   # Daily business metrics
    refresh_frequency: "1 hour"
    
  - name: historical_trends    # Reference data
    refresh_frequency: "24 hours"
```

---

## Implementation Checklist

### Pre-Implementation
- [ ] Business requirements gathering complete
- [ ] Data sources identified and accessible
- [ ] User roles and permissions defined
- [ ] Development environment set up

### Semantic Model Development
- [ ] Tables and relationships designed
- [ ] Dimensions vs facts properly classified
- [ ] Business descriptions and synonyms added
- [ ] Custom instructions defined
- [ ] Initial testing completed

### Integration Setup
- [ ] Cortex Search services configured (if needed)
- [ ] Verified Query Repository populated
- [ ] Monitoring and logging enabled
- [ ] Performance baselines established

### Deployment and Governance
- [ ] Production deployment completed
- [ ] User training conducted
- [ ] Monitoring dashboards operational
- [ ] Change management process established

### Ongoing Optimization
- [ ] Regular performance reviews scheduled
- [ ] User feedback collection process active
- [ ] VQR maintenance routine established
- [ ] Model iteration cycle defined

---

*This guide is based on official Snowflake Cortex Analyst documentation and real-world implementation best practices. For the most current information, refer to the [official Snowflake documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst).*
