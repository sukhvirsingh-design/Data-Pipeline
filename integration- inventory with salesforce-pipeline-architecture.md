# F&B Data Pipeline Architecture

## Overview

This document describes the data pipeline connecting point-of-sale transactions, inventory/COGS management, and centralized reporting for platform operations.

The goal is to give leadership a single, trustworthy view of financial performance by combining transaction-level revenue data with inventory-level cost data, rather than reconciling the two manually.

## Components

### 1. POS (e.g. Shift4)
Point-of-sale terminals capture individual transactions at the concession/venue level — items sold, quantities, timestamps, and payment method.

### 2. Payment gateway
Processes and settles POS transactions. Produces the financial settlement records (batch totals, card processing fees, settlement timing) that feed downstream reporting.

### 3. Yellow Dog software
Inventory management system tracking on-hand stock, receiving, depletion, and COGS. This is the source of truth for theoretical vs. actual variance reporting (pour cost, waste, shrinkage).

### 4. Integration layer
Middleware (e.g. MuleSoft, custom APIs, or ETL scripts) responsible for:
- Pulling settlement data from the payment gateway
- Pulling inventory/COGS data from Yellow Dog
- Transforming and reconciling both datasets into a common schema
- Running as a nightly batch sync into Salesforce

This is the layer where revenue (from the gateway) gets matched against cost (from Yellow Dog) to produce margin data.

### 5. Salesforce CRM (centralized data hub)
Receives the synced, reconciled data and acts as the system of record for reporting.

#### Salesforce Revenue Management (sub-module)
Built on top of the CRM data to provide:
- Overarching financial dashboards (revenue, cost, margin by location/event)
- Executive forecasting & analytics

## Data flow summary

```
POS terminals
   -> Payment gateway (financial settlement)
Yellow Dog (inventory / COGS)
   -> Integration layer (nightly batch ETL, reconciliation)
   -> Salesforce CRM
        -> Revenue Management (dashboards, forecasting)
```

Implementation Methods 

To execute this approach, businesses typically use one of three integration strategies:

Middleware/ETL Approach: Utilizing tools like MuleSoft (owned by Salesforce), Boomi, or Workato. These platforms extract the data from the gateway and Yellow Dog daily, transform the data into a matching format, and load it into Salesforce.

Custom API Integration: Developers write custom scripts utilizing REST APIs to automatically push summary data from the payment gateway and Yellow Dog directly into Salesforce custom objects at the end of every business day.

Data Lakehouse Mirroring: Large hospitality groups often dump raw logs from all three systems into a cloud data warehouse (like Snowflake or Salesforce Data Cloud). They then calculate the revenue there, using Salesforce purely to visualize the final reports.

## Open questions / next steps

- Confirm auth method between integration layer and each source system (API keys, OAuth)
- Define the reconciliation logic for matching settlement batches to inventory depletion periods
- Decide on sync frequency — nightly batch vs. near-real-time for high-volume event days
- Define variance reporting thresholds (theoretical vs. actual) that should trigger alerts in Salesforce
