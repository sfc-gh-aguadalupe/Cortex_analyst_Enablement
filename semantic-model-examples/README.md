# Semantic Model Examples

This directory contains ready-to-use semantic model YAML files for different industries and use cases. Each model is complete and can be deployed directly to Snowflake Cortex Analyst.

## Available Models

### Industry-Specific Models
- **[ecommerce-model.yaml](ecommerce-model.yaml)** - Complete e-commerce analytics model
- **[financial-services-model.yaml](financial-services-model.yaml)** - Banking and financial analytics
- **[healthcare-model.yaml](healthcare-model.yaml)** - Patient and provider analytics (HIPAA-compliant)
- **[manufacturing-model.yaml](manufacturing-model.yaml)** - Supply chain and production analytics
- **[saas-model.yaml](saas-model.yaml)** - SaaS subscription and usage analytics

### Model Features

Each semantic model includes:
- ✅ Complete table definitions with proper dimension/fact classification
- ✅ Comprehensive business descriptions and synonyms
- ✅ Verified Query Repository (VQR) with common business questions
- ✅ Custom instructions for business context
- ✅ Relationship definitions between tables
- ✅ Sample Cortex Search integrations where applicable

## How to Use These Models

### Step 1: Choose Your Industry Model
Select the model that best matches your business domain. You can also use these as starting points and customize for your specific needs.

### Step 2: Update Database References
Replace placeholder database/schema/table names with your actual Snowflake objects:

```yaml
# Change this:
base_table:
  database: your_database
  schema: your_schema  
  table: your_table

# To your actual values:
base_table:
  database: sales_db
  schema: public
  table: customers
```

### Step 3: Customize Business Logic
Update the custom instructions, descriptions, and synonyms to match your business terminology:

```yaml
# Update custom instructions
custom_instructions: |
  Your business context here:
  - Fiscal year definition
  - Customer segmentation rules
  - Revenue calculation methods
```

### Step 4: Deploy to Snowflake
1. **Via Snowsight UI**: Copy the YAML content into the semantic model creation interface
2. **Via API**: Use the Snowflake REST API to deploy programmatically
3. **Via SQL**: Create semantic views using the YAML content

### Step 5: Test and Validate
- Run the sample queries included in each model
- Verify relationships work correctly
- Test with your actual business questions

## Customization Guide

### Adding New Tables
```yaml
tables:
  - name: your_new_table
    base_table:
      database: your_database
      schema: your_schema
      table: your_new_table
    
    dimensions:
      - name: dimension_name
        expr: column_name
        description: "Business description"
        synonyms: ["synonym1", "synonym2"]
    
    facts:
      - name: fact_name
        expr: column_name
        description: "Measurable business metric"
```

### Adding Relationships
```yaml
relationships:
  - name: descriptive_relationship_name
    left_table: table1
    right_table: table2
    join_columns:
      - left_column: id_column
        right_column: id_column
    description: "Business meaning of this relationship"
```

### Adding Cortex Search Integration
```yaml
dimensions:
  - name: searchable_field
    expr: column_name
    description: "Field that benefits from semantic search"
    cortex_search_service:
      service: your_search_service_name
      literal_column: column_name
      database: your_database
      schema: your_schema
```

## Model Validation Checklist

Before deploying any model, ensure:

### Data Requirements
- [ ] All referenced tables exist and are accessible
- [ ] Primary key columns are marked as unique identifiers
- [ ] Join columns have matching data types
- [ ] No orphaned records that would break relationships

### Business Logic
- [ ] Descriptions accurately reflect business meaning
- [ ] Synonyms include terms your users actually say
- [ ] Custom instructions match your business rules
- [ ] Verified queries represent real business questions

### Technical Setup
- [ ] Database/schema/table names are correct
- [ ] User permissions are properly configured
- [ ] Warehouse is sized appropriately
- [ ] Cortex Search services are created (if used)

## Sample Business Questions

Each model includes verified queries for common business scenarios:

### E-commerce
- "What was our total revenue last month?"
- "Which products are our top sellers?"
- "Show customer acquisition trends by channel"

### Financial Services
- "What's our loan portfolio growth rate?"
- "Which branches have the highest deposit volumes?"
- "Show transaction trends by customer segment"

### Healthcare
- "What are the most common diagnoses this month?"
- "Show patient satisfaction scores by provider"
- "What's our average length of stay by department?"

### Manufacturing
- "What's our production efficiency by facility?"
- "Which suppliers have the best on-time delivery?"
- "Show quality metrics by product line"

### SaaS
- "What's our monthly recurring revenue growth?"
- "Which features have the highest adoption rates?"
- "Show churn risk by customer segment"

## Best Practices

### Performance Optimization
- Use materialized views for large datasets
- Implement appropriate clustering keys
- Create summary tables for common aggregations
- Right-size your warehouse for typical query complexity

### Data Quality
- Ensure consistent data types across join columns
- Handle null values appropriately in aggregations
- Exclude test/demo data from production models
- Implement data validation checks

### User Experience
- Use business-friendly terminology in descriptions
- Include comprehensive synonyms for natural language queries
- Provide clear custom instructions for context
- Build verified query libraries for common questions

## Support and Feedback

These models are based on:
- Official Snowflake Cortex Analyst documentation
- Real-world implementation best practices
- Common industry analytics patterns

For the most current information, always refer to the [official Snowflake documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst).

## Contributing

To improve these models:
1. Test with your real data and use cases
2. Document any issues or improvements needed
3. Share successful customizations and patterns
4. Provide feedback on business question coverage

---

*These semantic models provide a foundation for Cortex Analyst implementation. Customize them to match your specific business requirements and data structures.*
