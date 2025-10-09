# Cortex Analyst Best Practices

This guide provides official recommendations and proven strategies for implementing Cortex Analyst effectively in your organization.

## Table of Contents
1. [Semantic Model Design Best Practices](#semantic-model-design-best-practices)
2. [Verified Query Repository Best Practices](#verified-query-repository-best-practices)
3. [Cortex Search Integration Best Practices](#cortex-search-integration-best-practices)
4. [Team Collaboration and Governance](#team-collaboration-and-governance)

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
- [ ] Cortex Search services configured (if needed)
- [ ] Verified Query Repository populated
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

---

*This guide is based on official Snowflake Cortex Analyst documentation and real-world implementation best practices. For the most current information, refer to the [official Snowflake documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst).*
