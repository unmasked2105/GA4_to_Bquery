# GA4 → BigQuery Setup & Schema

Reference guide for configuring the **Google Analytics 4 → BigQuery export pipeline** and understanding the event schema used for analytics and downstream data processing.

This repository documents how to:

* Connect GA4 to BigQuery
* Understand the exported event schema
* Work with nested event parameters
* Query and transform GA4 data
* Build analytics-ready datasets

---

# Overview

Google Analytics 4 provides a **native export to BigQuery**, meaning no external ETL tools are required.

Once the connection is configured, GA4 automatically exports raw event data to BigQuery daily (or via streaming if enabled).

Typical pipeline:

```
GA4
  ↓
BigQuery Export Dataset
  ↓
SQL Transformations
  ↓
Analytics Tables
  ↓
BI / Data Science
```

---

# Tech Stack

This reference assumes the following stack:

* **Google Analytics 4**
* **Google BigQuery**
* **Google Cloud Platform**
* **Google Cloud Storage (optional)**
* **Databricks / SQL analytics layer**

---

# Prerequisites

Before configuring the export, ensure the following:

* Access to a **GA4 property**
* Access to **Google Cloud Platform**
* Permission to create datasets in **BigQuery**
* BigQuery API enabled in your GCP project

---

# Part 1 — Setting Up the GA4 → BigQuery Connection

GA4 includes a built-in export feature that automatically sends analytics data to BigQuery.

No additional ETL tools are required.

---

## 1. Confirm You Are Using GA4

BigQuery export is only available for **Google Analytics 4** properties.

Example identifiers:

| Property Type       | Example ID     |
| ------------------- | -------------- |
| GA4                 | `G-XXXXXXXXXX` |
| Universal Analytics | `UA-XXXXX-X`   |

Key points:

* GA4 → Native BigQuery export
* Universal Analytics → Requires workarounds

---

## 2. Enable BigQuery API

Inside Google Cloud Console:

```
APIs & Services
   → Library
      → BigQuery API
         → Enable
```

If your organization already uses BigQuery, this will likely already be enabled.

---

## 3. Link GA4 Property to BigQuery

Steps inside GA4:

```
Admin
  → Property
    → BigQuery Links
      → Link
```

Configuration options:

* Select **GCP project**
* Choose **dataset location** (US or EU)
* Enable **daily export**
* Optionally enable **streaming export**

Important:

The dataset region must match your existing BigQuery datasets to avoid cross-region limitations.

---

# Exported Dataset Structure

After linking GA4, BigQuery will automatically create a dataset with tables like:

```
analytics_XXXXXXXX
```

Daily tables:

```
events_YYYYMMDD
```

Example:

```
events_20260310
events_20260311
events_20260312
```

Streaming tables (if enabled):

```
events_intraday_YYYYMMDD
```

---

# GA4 Event Schema

GA4 exports data in an **event-based schema**.

Each row represents an event such as:

* `page_view`
* `session_start`
* `purchase`
* `scroll`

Important columns include:

| Field           | Description               |
| --------------- | ------------------------- |
| event_date      | Event date                |
| event_timestamp | Event timestamp           |
| event_name      | Event type                |
| user_pseudo_id  | Anonymous user identifier |
| event_params    | Nested parameters         |
| user_properties | Custom user attributes    |
| device          | Device metadata           |
| geo             | Geographic information    |
| traffic_source  | Marketing attribution     |

---

# Working With Nested Fields

GA4 stores parameters inside nested arrays.

Example field:

```
event_params
```

To access these values you must **UNNEST** the array.

Example query:

```sql
SELECT
  event_name,
  ep.key,
  ep.value.string_value
FROM `project.analytics_XXXX.events_*`,
UNNEST(event_params) AS ep
WHERE event_name = 'page_view'
```

This converts nested event parameters into queryable rows.

---

# Example Analytics Query

Page view count:

```sql
SELECT
  event_date,
  COUNT(*) AS pageviews
FROM `project.analytics_XXXX.events_*`
WHERE event_name = 'page_view'
GROUP BY event_date
ORDER BY event_date
```

---

# Typical Transformation Layer

Raw GA4 data usually requires transformation before analytics use.

Common models:

```
stg_ga4_events
fct_pageviews
fct_sessions
dim_users
```

These models help convert raw event data into:

* session-level metrics
* user-level analytics
* marketing attribution
* product analytics

---

# Best Practices

Recommended practices when working with GA4 in BigQuery:

* Partition tables by `event_date`
* Use wildcard tables (`events_*`)
* Filter early in queries to reduce scan cost
* Normalize repeated parameters
* Build reusable transformation models

---

# Architecture Example

```
GA4
 ↓
BigQuery Export (Raw Events)
 ↓
SQL Transformations
 ↓
Analytics Tables
 ↓
Dashboards / Data Science
```

---

# Repository Structure

Example project layout:

```
ga4-bigquery-setup/

docs/
  setup-guide.md
  schema.md
  queries.md

sql/
  transformations.sql
  analytics_models.sql

images/
  architecture.png
```

---

# Use Cases

This setup enables:

* product analytics
* marketing attribution
* funnel analysis
* retention analysis
* data science models
* custom dashboards

---

# License

MIT License

---

# Author

Created as a **data engineering reference for GA4 → BigQuery pipelines** used in analytics and internship projects.
