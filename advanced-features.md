# Advanced Cortex Analyst Features

This guide covers advanced Cortex Analyst capabilities based on official Snowflake documentation, including Verified Query Repository, Cortex Search integration, administrator monitoring, and custom instructions.

## Table of Contents
1. [Verified Query Repository (VQR)](#verified-query-repository-vqr)
2. [Cortex Search Integration](#cortex-search-integration)
3. [Administrator Monitoring and Observability](#administrator-monitoring-and-observability)
4. [Custom Instructions](#custom-instructions)
5. [Suggested Questions Feature](#suggested-questions-feature)
6. [REST API Integration](#rest-api-integration)
7. [Enterprise Implementation Patterns](#enterprise-implementation-patterns)

---

## Verified Query Repository (VQR)

Based on [official VQR documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst/verified-query-repository), the Verified Query Repository is a powerful feature that improves Cortex Analyst accuracy by storing validated question-answer pairs.

### Understanding VQR

The VQR serves as a learning mechanism for Cortex Analyst, where successful queries are stored and used to improve future responses. When users ask questions similar to verified queries, Cortex Analyst can provide more accurate and consistent results.

#### Key Benefits
- **Improved Accuracy**: Verified queries ensure consistent, correct responses
- **Faster Response Times**: Pre-validated queries execute more efficiently
- **Business Alignment**: Curated queries reflect actual business needs
- **User Training**: VQR serves as a library of effective query patterns

### VQR Implementation Strategy

#### Phase 1: Foundation Building
```yaml
# Start with core business questions
verified_queries:
  # Revenue fundamentals
  - question: "What was our total revenue last month?"
    verified_at: "2024-01-15T10:30:00Z"
  
  - question: "Show me revenue trends for the last 6 months"
    verified_at: "2024-01-15T10:30:00Z"
  
  # Customer basics
  - question: "How many customers do we have?"
    verified_at: "2024-01-15T10:30:00Z"
  
  - question: "How many new customers did we acquire this month?"
    verified_at: "2024-01-15T10:30:00Z"
```

#### Phase 2: Query Variations
```yaml
# Add multiple phrasings for the same business concept
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
```

#### Phase 3: Advanced Business Logic
```yaml
# Complex analytical questions
verified_queries:
  - question: "Compare this quarter's performance to the same quarter last year"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "Which customer segments have the highest lifetime value?"
    verified_at: "2024-01-15T10:30:00Z"
    
  - question: "Show me product performance by category with year-over-year growth"
    verified_at: "2024-01-15T10:30:00Z"
```

### VQR Maintenance Workflow

#### Identifying VQR Candidates
```sql
-- Query monitoring logs to find successful, frequently asked questions
SELECT 
  user_question,
  COUNT(*) as question_frequency,
  COUNT(DISTINCT user_name) as unique_users,
  AVG(CASE WHEN error_message IS NULL THEN 1 ELSE 0 END) as success_rate
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW', 
  'your_database.your_schema.your_semantic_view'
))
WHERE request_timestamp >= DATEADD('day', -30, CURRENT_TIMESTAMP())
  AND error_message IS NULL
GROUP BY user_question
HAVING question_frequency >= 5  -- Asked at least 5 times
  AND success_rate >= 0.9        -- High success rate
ORDER BY question_frequency DESC;
```

#### VQR Quality Control Process
1. **Validation**: Test each query thoroughly before adding to VQR
2. **Business Review**: Have business stakeholders confirm query accuracy
3. **Documentation**: Document the business logic behind each verified query
4. **Regular Audit**: Review VQR entries quarterly for relevance and accuracy

#### VQR Performance Monitoring
```sql
-- Monitor VQR effectiveness
WITH vqr_performance AS (
  SELECT 
    user_question,
    request_timestamp,
    CASE WHEN error_message IS NULL THEN 'Success' ELSE 'Error' END as result_status
  FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
    'SEMANTIC_VIEW', 
    'your_database.your_schema.your_semantic_view'
  ))
  WHERE user_question IN (
    -- List your VQR questions here
    'What was our total revenue last month?',
    'How many customers do we have?'
  )
  AND request_timestamp >= DATEADD('day', -7, CURRENT_TIMESTAMP())
)
SELECT 
  user_question,
  COUNT(*) as total_executions,
  SUM(CASE WHEN result_status = 'Success' THEN 1 ELSE 0 END) as successful_executions,
  (successful_executions / total_executions) * 100 as success_rate_percent
FROM vqr_performance
GROUP BY user_question
ORDER BY total_executions DESC;
```

---

## Cortex Search Integration

Based on [official Cortex Search documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst/cortex-analyst-search-integration), Cortex Search enhances Cortex Analyst's ability to find and match literal values in natural language queries.

### Understanding the Problem

Traditional exact-match searching fails when users ask questions like:
- "Show sales for iPhone" (when product names include "Apple iPhone 14 Pro Max 128GB")
- "Samsung phone revenue" (when database has specific model names)
- "Microsoft software sales" (when products include version numbers and SKUs)

### Cortex Search Solution

Cortex Search provides semantic similarity matching, finding relevant items even when exact text doesn't match.

#### Implementation Options

##### Option 1: UI-Based Setup (Recommended for Most Users)
1. **Navigate to Cortex Analyst UI**: AI & ML » Cortex Analyst » Create new model
2. **Select High-Cardinality Columns**: During dimension setup, choose text columns with many unique values
3. **Automatic Service Creation**: The UI creates and links Cortex Search services automatically
4. **Generated YAML**: The system generates the semantic model YAML with search integration

##### Option 2: Manual SQL Setup (Advanced Users)
```sql
-- Step 1: Create Cortex Search Service
CREATE OR REPLACE CORTEX SEARCH SERVICE product_name_search
  ON product_search_text
  WAREHOUSE = small
  TARGET_LAG = '1 hour'
  AS (
    SELECT DISTINCT 
      product_id,
      product_name as product_search_text,
      -- Include variations and synonyms
      brand || ' ' || product_name as brand_product_text,
      category || ' ' || subcategory as category_text
    FROM products
    WHERE product_name IS NOT NULL
    
    UNION ALL
    
    -- Add common search terms and synonyms
    SELECT 
      product_id,
      CASE 
        WHEN product_name ILIKE '%iphone%' THEN product_name || ' apple smartphone mobile phone'
        WHEN product_name ILIKE '%samsung%' THEN product_name || ' android smartphone mobile phone'
        WHEN product_name ILIKE '%laptop%' THEN product_name || ' computer notebook portable'
        ELSE product_name
      END as product_search_text,
      brand || ' ' || product_name as brand_product_text,
      category || ' ' || subcategory as category_text
    FROM products
    WHERE product_name IS NOT NULL
  );
```

```yaml
# Step 2: Link to Semantic Model
tables:
  - name: products
    base_table:
      database: your_database
      schema: your_schema
      table: products
    
    dimensions:
      - name: product_name
        expr: product_name
        description: "Product names including brand, model, and specifications"
        synonyms: ["product", "item", "SKU", "merchandise"]
        cortex_search_service:
          service: product_name_search
          literal_column: product_name
          database: your_database
          schema: your_schema
```

### Advanced Search Integration Patterns

#### Multi-Column Search Enhancement
```sql
-- Create comprehensive search service covering multiple attributes
CREATE OR REPLACE CORTEX SEARCH SERVICE comprehensive_product_search
  ON search_content
  WAREHOUSE = medium
  TARGET_LAG = '2 hours'
  AS (
    SELECT DISTINCT
      product_id,
      -- Combine multiple searchable fields
      product_name || ' ' || 
      COALESCE(brand, '') || ' ' || 
      COALESCE(category, '') || ' ' || 
      COALESCE(subcategory, '') || ' ' ||
      COALESCE(description, '') as search_content
    FROM products
    WHERE product_name IS NOT NULL
  );
```

#### Industry-Specific Search Examples

##### Healthcare: Diagnosis and Procedure Search
```sql
CREATE OR REPLACE CORTEX SEARCH SERVICE medical_diagnosis_search
  ON diagnosis_search_text
  WAREHOUSE = small
  TARGET_LAG = '4 hours'
  AS (
    SELECT DISTINCT
      diagnosis_code,
      diagnosis_description || ' ' || 
      COALESCE(category, '') || ' ' ||
      -- Include common medical synonyms
      CASE 
        WHEN diagnosis_description ILIKE '%diabetes%' THEN diagnosis_description || ' blood sugar glucose'
        WHEN diagnosis_description ILIKE '%hypertension%' THEN diagnosis_description || ' high blood pressure'
        WHEN diagnosis_description ILIKE '%myocardial infarction%' THEN diagnosis_description || ' heart attack MI'
        ELSE diagnosis_description
      END as diagnosis_search_text
    FROM medical_diagnoses
    WHERE diagnosis_description IS NOT NULL
  );
```

##### Financial Services: Transaction Category Search
```sql
CREATE OR REPLACE CORTEX SEARCH SERVICE transaction_category_search
  ON category_search_text
  WAREHOUSE = xsmall
  TARGET_LAG = '1 hour'
  AS (
    SELECT DISTINCT
      category_code,
      category_name || ' ' ||
      -- Add financial synonyms
      CASE 
        WHEN category_name ILIKE '%restaurant%' THEN category_name || ' dining food meal'
        WHEN category_name ILIKE '%gas%' THEN category_name || ' fuel gasoline petroleum'
        WHEN category_name ILIKE '%grocery%' THEN category_name || ' food supermarket market'
        ELSE category_name
      END as category_search_text
    FROM transaction_categories
    WHERE category_name IS NOT NULL
  );
```

### Search Service Optimization

#### Performance Tuning
```sql
-- Monitor search service performance
SELECT 
  SERVICE_NAME,
  TARGET_LAG,
  LAST_REFRESH_TIME,
  REFRESH_STATUS,
  TOTAL_DOCS,
  WAREHOUSE_SIZE
FROM INFORMATION_SCHEMA.CORTEX_SEARCH_SERVICES
WHERE SERVICE_NAME LIKE '%your_service%';
```

#### Warehouse Sizing Guidelines
```sql
-- Right-size warehouse based on data volume
-- XS: < 100K records
-- S:  100K - 1M records  
-- M:  1M - 10M records
-- L:  10M+ records

ALTER CORTEX SEARCH SERVICE your_search_service 
SET WAREHOUSE = 'medium',
    TARGET_LAG = '30 minutes';
```

#### Search Quality Testing
```sql
-- Test search effectiveness
SELECT 
  search_term,
  PARSE_JSON(search_result):content::STRING as matched_content,
  PARSE_JSON(search_result):score::FLOAT as relevance_score
FROM (
  SELECT 
    'iphone' as search_term,
    result as search_result
  FROM TABLE(your_database.your_schema.product_name_search('iphone'))
  
  UNION ALL
  
  SELECT 
    'samsung phone' as search_term,
    result as search_result
  FROM TABLE(your_database.your_schema.product_name_search('samsung phone'))
)
ORDER BY search_term, relevance_score DESC;
```

---

## Administrator Monitoring and Observability

Based on [official monitoring documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst/admin-observability), comprehensive monitoring is essential for maintaining and optimizing Cortex Analyst performance.

### Monitoring Architecture

#### Data Flow
```
User Query → Cortex Analyst → SQL Generation → Execution → Results
     ↓              ↓              ↓            ↓          ↓
Event Logging → Request Tracking → Performance Monitoring → Analytics
```

#### Log Data Structure
The monitoring system captures:
- User who asked the question
- Original natural language question
- Generated SQL query
- Execution results or error messages
- Request and response metadata
- Performance timing information

### Accessing Monitoring Data

#### Method 1: Snowsight UI
Navigate to your semantic model and use the **Monitoring** tab to view:
- Query volume trends
- Success/failure rates
- Popular questions
- User activity patterns

#### Method 2: SQL Queries
```sql
-- Basic monitoring query
SELECT 
  user_name,
  user_question,
  generated_sql,
  error_message,
  request_timestamp
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW',
  'your_database.your_schema.your_semantic_view'
))
WHERE request_timestamp >= DATEADD('day', -7, CURRENT_TIMESTAMP())
ORDER BY request_timestamp DESC;
```

#### Method 3: Administrator Views (Requires Special Role)
```sql
-- Organization-wide monitoring (requires CORTEX_ANALYST_REQUESTS_ADMIN role)
SELECT 
  semantic_model_name,
  user_name,
  user_question,
  success_indicator,
  request_timestamp
FROM SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS_V
WHERE request_timestamp >= DATEADD('day', -30, CURRENT_TIMESTAMP())
ORDER BY request_timestamp DESC;
```

### Key Monitoring Metrics

#### 1. Usage and Adoption Metrics
```sql
-- Daily active users and query volume
CREATE OR REPLACE VIEW cortex_analyst_usage_metrics AS
SELECT 
  DATE_TRUNC('day', request_timestamp) as metric_date,
  COUNT(*) as total_queries,
  COUNT(DISTINCT user_name) as unique_users,
  AVG(LENGTH(user_question)) as avg_question_length,
  COUNT(DISTINCT DATE_TRUNC('hour', request_timestamp)) as active_hours
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW',
  'your_database.your_schema.your_semantic_view'
))
WHERE request_timestamp >= DATEADD('day', -30, CURRENT_TIMESTAMP())
GROUP BY DATE_TRUNC('day', request_timestamp)
ORDER BY metric_date DESC;
```

#### 2. Quality and Accuracy Metrics
```sql
-- Success rates and error analysis
CREATE OR REPLACE VIEW cortex_analyst_quality_metrics AS
SELECT 
  DATE_TRUNC('day', request_timestamp) as metric_date,
  COUNT(*) as total_queries,
  COUNT(CASE WHEN error_message IS NULL THEN 1 END) as successful_queries,
  COUNT(CASE WHEN error_message IS NOT NULL THEN 1 END) as failed_queries,
  (successful_queries / total_queries) * 100 as success_rate_percent,
  
  -- Common error categories
  COUNT(CASE WHEN error_message ILIKE '%permission%' THEN 1 END) as permission_errors,
  COUNT(CASE WHEN error_message ILIKE '%timeout%' THEN 1 END) as timeout_errors,
  COUNT(CASE WHEN error_message ILIKE '%syntax%' THEN 1 END) as syntax_errors
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW',
  'your_database.your_schema.your_semantic_view'
))
WHERE request_timestamp >= DATEADD('day', -30, CURRENT_TIMESTAMP())
GROUP BY DATE_TRUNC('day', request_timestamp)
ORDER BY metric_date DESC;
```

#### 3. Performance Metrics
```sql
-- Query complexity and performance analysis
CREATE OR REPLACE VIEW cortex_analyst_performance_metrics AS
SELECT 
  DATE_TRUNC('hour', request_timestamp) as metric_hour,
  COUNT(*) as query_count,
  AVG(LENGTH(generated_sql)) as avg_sql_complexity,
  MAX(LENGTH(generated_sql)) as max_sql_complexity,
  
  -- Classify queries by complexity
  COUNT(CASE WHEN LENGTH(generated_sql) < 500 THEN 1 END) as simple_queries,
  COUNT(CASE WHEN LENGTH(generated_sql) BETWEEN 500 AND 1500 THEN 1 END) as medium_queries,
  COUNT(CASE WHEN LENGTH(generated_sql) > 1500 THEN 1 END) as complex_queries
FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
  'SEMANTIC_VIEW',
  'your_database.your_schema.your_semantic_view'
))
WHERE request_timestamp >= DATEADD('day', -7, CURRENT_TIMESTAMP())
  AND error_message IS NULL
GROUP BY DATE_TRUNC('hour', request_timestamp)
ORDER BY metric_hour DESC;
```

### Automated Monitoring and Alerting

#### Daily Health Check Task
```sql
CREATE OR REPLACE TASK cortex_analyst_daily_health_check
  WAREHOUSE = monitoring_wh
  SCHEDULE = 'USING CRON 0 8 * * * UTC'  -- Daily at 8 AM UTC
AS
DECLARE
  yesterday_success_rate FLOAT;
  success_threshold FLOAT DEFAULT 85.0;
  min_query_volume INT DEFAULT 10;
  yesterday_volume INT;
BEGIN
  -- Calculate yesterday's metrics
  SELECT 
    (COUNT(CASE WHEN error_message IS NULL THEN 1 END) / COUNT(*)) * 100,
    COUNT(*)
  INTO yesterday_success_rate, yesterday_volume
  FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
    'SEMANTIC_VIEW',
    'your_database.your_schema.your_semantic_view'
  ))
  WHERE DATE_TRUNC('day', request_timestamp) = CURRENT_DATE - 1;
  
  -- Check for issues and log alerts
  IF (yesterday_success_rate < success_threshold AND yesterday_volume >= min_query_volume) THEN
    INSERT INTO monitoring.cortex_analyst_alerts (
      alert_date,
      alert_type,
      metric_value,
      threshold_value,
      description
    ) VALUES (
      CURRENT_DATE - 1,
      'LOW_SUCCESS_RATE',
      yesterday_success_rate,
      success_threshold,
      'Daily success rate below threshold'
    );
  END IF;
  
  IF (yesterday_volume = 0) THEN
    INSERT INTO monitoring.cortex_analyst_alerts (
      alert_date,
      alert_type,
      metric_value,
      threshold_value,
      description
    ) VALUES (
      CURRENT_DATE - 1,
      'NO_USAGE',
      yesterday_volume,
      min_query_volume,
      'No queries detected for the day'
    );
  END IF;
END;

-- Start the monitoring task
ALTER TASK cortex_analyst_daily_health_check RESUME;
```

#### User Adoption Monitoring
```sql
-- Monitor user onboarding and adoption
CREATE OR REPLACE VIEW user_adoption_analysis AS
WITH user_activity AS (
  SELECT 
    user_name,
    MIN(request_timestamp) as first_query_date,
    MAX(request_timestamp) as last_query_date,
    COUNT(*) as total_queries,
    COUNT(DISTINCT DATE_TRUNC('day', request_timestamp)) as active_days,
    AVG(CASE WHEN error_message IS NULL THEN 1 ELSE 0 END) as personal_success_rate
  FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
    'SEMANTIC_VIEW',
    'your_database.your_schema.your_semantic_view'
  ))
  WHERE request_timestamp >= DATEADD('day', -90, CURRENT_TIMESTAMP())
  GROUP BY user_name
)
SELECT 
  user_name,
  first_query_date,
  last_query_date,
  total_queries,
  active_days,
  ROUND(total_queries / active_days, 2) as avg_queries_per_day,
  ROUND(personal_success_rate * 100, 2) as success_rate_percent,
  DATEDIFF('day', first_query_date, CURRENT_DATE()) as days_since_first_use,
  DATEDIFF('day', last_query_date, CURRENT_DATE()) as days_since_last_use,
  
  -- User engagement classification
  CASE 
    WHEN total_queries >= 50 AND active_days >= 20 THEN 'Power User'
    WHEN total_queries >= 20 AND active_days >= 10 THEN 'Regular User'
    WHEN total_queries >= 5 AND active_days >= 3 THEN 'Occasional User'
    WHEN total_queries < 5 THEN 'New/Trial User'
    ELSE 'Inactive User'
  END as user_category
FROM user_activity
ORDER BY total_queries DESC;
```

---

## Custom Instructions

Custom instructions allow you to fine-tune Cortex Analyst's behavior to match your specific business context, terminology, and analytical preferences.

### Understanding Custom Instructions

Custom instructions serve as business context and analytical guidelines that help Cortex Analyst:
- Understand your specific business terminology
- Apply consistent calculation methods
- Follow your data interpretation rules
- Use appropriate time periods and filters

### Implementation Categories

#### 1. Business Context Instructions
```yaml
custom_instructions: |
  Business Context for Financial Services:
  
  ## Regulatory Environment
  - All financial calculations must comply with Basel III standards
  - Risk-weighted assets calculated using standardized approach
  - Capital ratios reported according to Federal Reserve guidelines
  - Interest rate risk measured using duration analysis
  
  ## Business Calendar
  - Fiscal year aligns with calendar year (January-December)
  - Quarterly reporting periods end March 31, June 30, September 30, December 31
  - Month-end cutoff is last business day of month
  - Holiday adjustments apply to business day calculations
  
  ## Customer Classification
  - Retail banking: Individual customers and small businesses
  - Commercial banking: Mid-market businesses ($10M-$500M revenue)
  - Corporate banking: Large enterprises (>$500M revenue)
  - Private banking: High net worth individuals (>$1M investable assets)
```

#### 2. Technical Calculation Rules
```yaml
custom_instructions: |
  Technical Calculation Standards:
  
  ## Financial Calculations
  - Interest calculations use actual/365 day count convention
  - Currency conversions use end-of-day rates unless specified
  - All percentages rounded to 2 decimal places for reporting
  - Net present value calculations use risk-free rate + credit spread
  
  ## Data Quality Rules
  - Exclude test accounts (account_type != 'TEST')
  - Filter out inactive accounts (status = 'ACTIVE')
  - Use settlement date for transaction analysis
  - Handle null values by excluding from calculations
  
  ## Performance Metrics
  - Return on assets = net income / average total assets
  - Efficiency ratio = non-interest expense / (net interest income + non-interest income)
  - Loan loss provision coverage = allowance / total loans
  - Net interest margin = (interest income - interest expense) / average earning assets
```

#### 3. Industry-Specific Guidelines

##### Healthcare Custom Instructions
```yaml
custom_instructions: |
  Healthcare Analytics Guidelines:
  
  ## Privacy and Compliance
  - All patient data must be aggregated (minimum cell size: 11 patients)
  - Use age groups instead of specific ages: 0-17, 18-35, 36-55, 56-75, 75+
  - Geographic data limited to state level for privacy
  - Exclude mental health and substance abuse data from general reports
  
  ## Clinical Standards
  - Primary diagnosis takes precedence over secondary diagnoses
  - Length of stay excludes same-day visits
  - Readmission window is 30 days from discharge
  - Mortality rates risk-adjusted using standard methodology
  
  ## Quality Metrics
  - Patient satisfaction scores include response rate context
  - Provider performance requires minimum 20 cases for significance
  - Infection rates follow CDC definitions
  - Quality measures use rolling 12-month periods
  
  ## Time Periods
  - Clinical year follows calendar year
  - "Recent" refers to last 90 days for clinical patterns
  - Quality reporting uses fiscal year (October-September)
  - Seasonal adjustments for respiratory illnesses
```

##### Manufacturing Custom Instructions
```yaml
custom_instructions: |
  Manufacturing Operations Guidelines:
  
  ## Production Standards
  - Overall Equipment Effectiveness (OEE) = Availability × Performance × Quality
  - First-pass yield = units passing quality first time / total units produced
  - Cycle time includes setup, processing, and teardown
  - Planned downtime excludes maintenance from availability calculations
  
  ## Quality Management
  - Defect rate calculated as defects per million opportunities (DPMO)
  - Six Sigma quality target: <3.4 DPMO
  - Critical quality characteristics defined per product specification
  - Root cause analysis required for defect rates >1%
  
  ## Cost Accounting
  - Standard cost includes material, labor, and overhead allocation
  - Variance analysis compares actual vs standard costs
  - Scrap costs tracked separately from normal production costs
  - Energy allocation based on machine utilization hours
  
  ## Shift Operations
  - 1st shift: 6:00 AM - 2:00 PM
  - 2nd shift: 2:00 PM - 10:00 PM  
  - 3rd shift: 10:00 PM - 6:00 AM
  - Weekend premium rates apply Saturday-Sunday
```

### Advanced Custom Instructions Patterns

#### Conditional Logic Instructions
```yaml
custom_instructions: |
  Conditional Analysis Rules:
  
  ## Seasonal Business Adjustments
  - Q4 analysis should compare to previous Q4 (holiday seasonality)
  - Summer months (June-August) exclude seasonal products from trends
  - Back-to-school period (August-September) analyzed separately
  - Holiday sales (November-December) normalized for shopping days
  
  ## Geographic Analysis
  - East Coast markets close at 4 PM ET, West Coast at 1 PM ET
  - International sales converted to USD using month-end rates
  - Regional analysis groups: Northeast, Southeast, Midwest, Southwest, West
  - Urban vs suburban classifications based on population density
  
  ## Customer Lifecycle Analysis
  - New customers: first purchase within last 90 days
  - Loyal customers: active for >2 years with regular purchases
  - At-risk customers: no purchase in last 60 days
  - Winback customers: returned after >180 day absence
```

#### Error Handling Instructions
```yaml
custom_instructions: |
  Data Quality and Error Handling:
  
  ## Missing Data Treatment
  - For revenue analysis: exclude records with null amounts
  - For customer metrics: use last known value if recent (<30 days)
  - For trend analysis: interpolate missing months if <2 consecutive gaps
  - For geographic analysis: classify unknown locations as "Other"
  
  ## Outlier Detection
  - Flag transactions >10x average as potential outliers
  - Exclude test customers (email domains: @test.com, @example.com)
  - Remove negative quantities unless specifically refunds/returns
  - Cap extreme values at 99th percentile for statistical analysis
  
  ## Validation Rules
  - Cross-check totals against source system reconciliation
  - Verify date ranges are within business operational periods
  - Ensure customer counts match CRM system within 5% tolerance
  - Validate currency amounts are positive unless specifically credits
```

### Custom Instructions Best Practices

#### 1. Start Simple and Evolve
```yaml
# Version 1.0 - Basic rules
custom_instructions: |
  Basic Business Rules:
  - Fiscal year starts February 1
  - Customer segments: Premium (>$5K), Standard ($1K-$5K), Basic (<$1K)
  - Round financial amounts to 2 decimal places

# Version 2.0 - Add complexity based on usage
custom_instructions: |
  Enhanced Business Rules:
  - Fiscal year starts February 1, quarters: Feb-Apr, May-Jul, Aug-Oct, Nov-Jan
  - Customer segments: Premium (>$5K annual), Standard ($1K-$5K), Basic (<$1K)
  - New customer: first purchase within 90 days
  - Seasonal products: holiday (Nov-Dec), summer (Jun-Aug), back-to-school (Aug-Sep)
  - Round financial amounts to 2 decimal places
  - Exclude test accounts (email ending @test.com)
  - International sales converted using month-end exchange rates
```

#### 2. Document Business Logic
```yaml
custom_instructions: |
  Customer Lifetime Value Calculation:
  
  ## Methodology
  CLV = (Average Monthly Revenue × Gross Margin %) ÷ Monthly Churn Rate
  
  ## Components
  - Average Monthly Revenue: Sum of monthly revenue / number of months active
  - Gross Margin %: (Revenue - Cost of Goods Sold) / Revenue
  - Monthly Churn Rate: Customers lost in month / customers at start of month
  
  ## Data Sources
  - Revenue: order_total from orders table
  - COGS: cost_of_goods_sold from order_items × products join
  - Customer status: derived from last_order_date in customer table
  
  ## Business Rules
  - Include only completed orders (status = 'DELIVERED')
  - Exclude refunds and returns from revenue calculation
  - Minimum 6 months of history required for CLV calculation
  - Update calculations monthly on the 5th business day
```

---

## Suggested Questions Feature

The Suggested Questions feature helps users discover what they can ask Cortex Analyst by providing relevant, contextual question examples.

### Understanding Suggested Questions

Suggested Questions serve multiple purposes:
- **User Onboarding**: Help new users understand capabilities
- **Feature Discovery**: Show advanced analytical possibilities  
- **Query Inspiration**: Provide starting points for analysis
- **Best Practice Sharing**: Demonstrate effective question patterns

### Implementation Strategies

#### 1. Category-Based Suggestions
```yaml
# Organize suggestions by business function
suggested_questions:
  revenue_analysis:
    - "What was our revenue last month?"
    - "Show revenue trends by product category"
    - "Compare this quarter's revenue to last quarter"
    
  customer_insights:
    - "How many new customers did we acquire?"
    - "Which customer segments are most valuable?"
    - "Show customer retention rates by cohort"
    
  product_performance:
    - "What are our top-selling products?"
    - "Which products have declining sales?"
    - "Show product performance by region"
```

#### 2. User Role-Based Suggestions
```yaml
# Different suggestions for different user types
sales_team_questions:
  - "Show my team's performance this quarter"
  - "Which prospects are most likely to convert?"
  - "What's our win rate by deal size?"
  
marketing_team_questions:
  - "Which campaigns generated the most leads?"
  - "What's our cost per acquisition by channel?"
  - "Show conversion rates by traffic source"
  
executive_questions:
  - "What are our key performance indicators?"
  - "Show year-over-year growth trends"
  - "Which business segments are most profitable?"
```

#### 3. Progressive Complexity Suggestions
```yaml
# Start simple, offer more complex options
beginner_questions:
  - "How many customers do we have?"
  - "What was our revenue last month?"
  - "Show our top 10 products"
  
intermediate_questions:
  - "Compare revenue by customer segment"
  - "Show monthly trends for the last year"
  - "Which regions are growing fastest?"
  
advanced_questions:
  - "Calculate customer lifetime value by acquisition channel"
  - "Show cohort retention analysis"
  - "Analyze seasonal patterns with year-over-year growth"
```

### Dynamic Suggestion Generation

#### Context-Aware Suggestions
```sql
-- Generate suggestions based on user's recent activity
WITH user_query_patterns AS (
  SELECT 
    user_name,
    ARRAY_AGG(user_question) as recent_questions,
    COUNT(*) as query_count
  FROM TABLE(SNOWFLAKE.LOCAL.CORTEX_ANALYST_REQUESTS(
    'SEMANTIC_VIEW',
    'your_database.your_schema.your_semantic_view'
  ))
  WHERE user_name = CURRENT_USER()
    AND request_timestamp >= DATEADD('day', -7, CURRENT_TIMESTAMP())
  GROUP BY user_name
)
SELECT 
  CASE 
    WHEN ARRAY_CONTAINS('revenue'::VARIANT, recent_questions) THEN 
      'Try asking about revenue by different dimensions like region or product'
    WHEN ARRAY_CONTAINS('customer'::VARIANT, recent_questions) THEN
      'Explore customer segmentation and lifetime value analysis'
    ELSE
      'Start with basic questions about revenue, customers, or products'
  END as personalized_suggestion
FROM user_query_patterns;
```

---

## REST API Integration

Cortex Analyst provides REST API endpoints for programmatic integration into applications, dashboards, and automated workflows.

### API Architecture

#### Authentication
```python
import requests

# Use Snowflake JWT token or OAuth for authentication
headers = {
    "Authorization": f"Bearer {jwt_token}",
    "Content-Type": "application/json",
    "Accept": "application/json"
}
```

#### Basic API Call Structure
```python
def query_cortex_analyst(question, semantic_model, account_url, token):
    """
    Send natural language question to Cortex Analyst
    """
    endpoint = f"{account_url}/api/v2/cortex/analyst/message"
    
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
    
    headers = {
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json"
    }
    
    response = requests.post(endpoint, headers=headers, json=payload)
    return response.json()
```

### Enterprise Integration Patterns

#### 1. Dashboard Integration
```python
class CortexAnalystDashboard:
    def __init__(self, account_url, token, semantic_model):
        self.account_url = account_url
        self.token = token
        self.semantic_model = semantic_model
        self.base_url = f"{account_url}/api/v2/cortex/analyst"
    
    def get_kpi_data(self, kpi_questions):
        """Fetch multiple KPIs for dashboard display"""
        results = {}
        
        for kpi_name, question in kpi_questions.items():
            try:
                response = self.query_cortex_analyst(question)
                results[kpi_name] = {
                    "value": self.extract_metric_value(response),
                    "sql": response.get("generated_sql"),
                    "success": True
                }
            except Exception as e:
                results[kpi_name] = {
                    "error": str(e),
                    "success": False
                }
        
        return results
    
    def extract_metric_value(self, response):
        """Extract single metric value from response"""
        # Implementation depends on response format
        data = response.get("data", [])
        if data and len(data) > 0:
            return data[0][0]  # First row, first column
        return None

# Usage example
dashboard = CortexAnalystDashboard(account_url, token, semantic_model)

kpis = {
    "monthly_revenue": "What was our revenue last month?",
    "new_customers": "How many new customers did we acquire this month?",
    "avg_order_value": "What's our average order value this quarter?"
}

kpi_results = dashboard.get_kpi_data(kpis)
```

#### 2. Automated Reporting
```python
class AutomatedReporting:
    def __init__(self, cortex_client):
        self.cortex_client = cortex_client
        
    def generate_weekly_report(self):
        """Generate automated weekly business report"""
        report_questions = [
            "What was our revenue this week vs last week?",
            "Which products were top sellers this week?",
            "How many new customers did we acquire?",
            "What's our customer satisfaction score this week?"
        ]
        
        report_data = {}
        for question in report_questions:
            result = self.cortex_client.query(question)
            report_data[question] = result
        
        # Generate report document
        return self.format_report(report_data)
    
    def schedule_reports(self):
        """Set up scheduled report generation"""
        # Integration with task scheduler (cron, Airflow, etc.)
        pass
```

#### 3. Chatbot Integration
```python
class BusinessIntelligenceChatbot:
    def __init__(self, cortex_client):
        self.cortex_client = cortex_client
        self.conversation_history = []
    
    def handle_user_question(self, user_question):
        """Process user question and return business insights"""
        try:
            # Validate question appropriateness
            if self.is_valid_business_question(user_question):
                response = self.cortex_client.query(user_question)
                
                # Format response for chat interface
                formatted_response = self.format_chat_response(response)
                
                # Store conversation history
                self.conversation_history.append({
                    "question": user_question,
                    "response": formatted_response,
                    "timestamp": datetime.now()
                })
                
                return formatted_response
            else:
                return "I can help with business questions about revenue, customers, products, and operations."
                
        except Exception as e:
            return f"I encountered an error: {str(e)}"
    
    def suggest_follow_up_questions(self, current_question):
        """Suggest related questions based on current context"""
        # Implementation would analyze current question and suggest related queries
        pass
```

---

## Enterprise Implementation Patterns

### Multi-Tenant Architecture

#### 1. Semantic Model Isolation
```yaml
# Customer A semantic model
name: customer_a_analytics
description: "Analytics for Customer A with isolated data access"

tables:
  - name: customer_a_sales
    base_table:
      database: tenant_a_db
      schema: sales
      table: orders
    # ... rest of configuration

# Security context
custom_instructions: |
  Data Isolation Rules:
  - Only access data for tenant_id = 'CUSTOMER_A'
  - All queries automatically filtered by tenant
  - Cross-tenant data access prohibited
```

#### 2. Row-Level Security Integration
```sql
-- Create row-level security policy
CREATE OR REPLACE ROW ACCESS POLICY tenant_isolation_policy AS (tenant_id VARCHAR) 
RETURNS BOOLEAN ->
  CURRENT_USER_TENANT() = tenant_id
;

-- Apply to tables
ALTER TABLE orders ADD ROW ACCESS POLICY tenant_isolation_policy ON (tenant_id);

-- Semantic model automatically inherits RLS
```

### High Availability Configuration

#### 1. Multi-Region Deployment
```yaml
# Primary region semantic model
primary_model:
  name: global_analytics_primary
  region: us-east-1
  database: analytics_primary
  
# Secondary region semantic model  
secondary_model:
  name: global_analytics_secondary
  region: us-west-2
  database: analytics_replica
  
# Failover configuration
failover:
  detection_interval: 30s
  automatic_failover: true
  failback_policy: manual
```

#### 2. Load Balancing Strategy
```python
class CortexAnalystLoadBalancer:
    def __init__(self, primary_endpoint, secondary_endpoints):
        self.primary = primary_endpoint
        self.secondaries = secondary_endpoints
        self.health_check_interval = 60  # seconds
        
    def route_query(self, question, semantic_model):
        """Route query to best available endpoint"""
        # Try primary first
        if self.is_healthy(self.primary):
            return self.query_endpoint(self.primary, question, semantic_model)
        
        # Fallback to secondary endpoints
        for endpoint in self.secondaries:
            if self.is_healthy(endpoint):
                return self.query_endpoint(endpoint, question, semantic_model)
        
        raise Exception("No healthy endpoints available")
    
    def is_healthy(self, endpoint):
        """Check endpoint health"""
        try:
            response = requests.get(f"{endpoint}/health", timeout=5)
            return response.status_code == 200
        except:
            return False
```

### Performance Optimization at Scale

#### 1. Query Result Caching
```python
class CortexAnalystCache:
    def __init__(self, redis_client, cache_ttl=3600):
        self.redis = redis_client
        self.ttl = cache_ttl
        
    def get_cached_result(self, question, semantic_model):
        """Retrieve cached query result"""
        cache_key = self.generate_cache_key(question, semantic_model)
        cached_result = self.redis.get(cache_key)
        
        if cached_result:
            return json.loads(cached_result)
        return None
    
    def cache_result(self, question, semantic_model, result):
        """Store query result in cache"""
        cache_key = self.generate_cache_key(question, semantic_model)
        self.redis.setex(
            cache_key, 
            self.ttl, 
            json.dumps(result)
        )
    
    def generate_cache_key(self, question, semantic_model):
        """Generate unique cache key"""
        content = f"{question}:{semantic_model}"
        return hashlib.md5(content.encode()).hexdigest()
```

#### 2. Query Optimization Pipeline
```python
class QueryOptimizer:
    def __init__(self, cortex_client):
        self.cortex_client = cortex_client
        self.optimization_rules = [
            self.add_time_filters,
            self.optimize_aggregations,
            self.limit_result_sets
        ]
    
    def optimize_question(self, question):
        """Apply optimization rules to question"""
        optimized = question
        
        for rule in self.optimization_rules:
            optimized = rule(optimized)
            
        return optimized
    
    def add_time_filters(self, question):
        """Add appropriate time filters for performance"""
        if "revenue" in question.lower() and "last" not in question.lower():
            return f"{question} for the last 12 months"
        return question
    
    def limit_result_sets(self, question):
        """Add limits for large result sets"""
        if "show all" in question.lower():
            return question.replace("show all", "show top 100")
        return question
```

---

*This advanced features guide is based on official Snowflake Cortex Analyst documentation. For the most current API specifications and advanced configurations, refer to the [official Snowflake documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst).*
