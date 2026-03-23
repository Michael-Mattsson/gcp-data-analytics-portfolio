# Manage Data Models in Looker: Challenge Lab

This document outlines the implementation of key LookML concepts including metric creation, model optimization, data governance, and usability improvements.

---

## Task 1 — Create LookML Objects

**What this solves:**  
Fixes missing foundational metrics required for a partially implemented aggregate table and ensures calculations (profit, revenue) are reusable and consistent.

### File: `views/order_items.view.lkml`

**Add Dimension and Measure**
```lookml
dimension: profit {
  label: "Profit"
  description: "Profit for each item"
  type: number
  sql: ${sale_price} - ${products.cost} ;;
  # Alternative: ${inventory_items.cost}
  value_format_name: usd
}

measure: total_profit {
  label: "Total Profit"
  description: "Total profit across items"
  type: sum
  sql: ${profit} ;;
  value_format_name: usd
}
```

### File: `models/training_ecommerce.model.lkml`

**Update Datagroup**
```lookml
datagroup: weekly_datagroup_Fypl {
  # Caching policy (name provided by lab)
  # sql_trigger: SELECT MAX(id) FROM etl_log ;;
  max_cache_age: "168 hours"
}

persist_with: weekly_datagroup_Fypl
```

---

## Task 2 — Create and Fix a Refinement with an Aggregate Table

**What this solves:**  
Repairs a broken aggregate table so Looker can use precomputed weekly data for faster queries and improved performance.

### File: `models/training_ecommerce.model.lkml`

```lookml
explore: +order_items {
  label: "Order Items - Aggregate Profit and Revenue"

  aggregate_table: weekly_aggregate_revenue_profit {
    query: {
      dimensions: [order_items.created_date]
      measures: [order_items.total_revenue, order_items.total_profit]
    }

    materialization: {
      datagroup_trigger: weekly_datagroup_3JJi
      increment_key: "created_date"
    }
  }
}
```

---

## Task 3 — Extend a View (Handling PII Data)

**What this solves:**  
Separates sensitive user data (PII) into a dedicated, extendable view to support controlled access and better data governance.

### File: `views/user_pii_challenge_ltzq.view.lkml`

```lookml
view: user_pii_challenge_ltzq {
  extension: required

  dimension: first_name {
    hidden: yes
    type: string
    sql: ${TABLE}.first_name ;;
  }

  dimension: last_name {
    hidden: yes
    type: string
    sql: ${TABLE}.last_name ;;
  }

  dimension: email {
    hidden: yes
    type: string
    sql: ${TABLE}.email ;;
  }

  dimension: id {
    primary_key: yes
    hidden: yes
    sql: ${TABLE}.id ;;
  }

  dimension: latitude {
    hidden: yes
    type: number
    sql: ${TABLE}.latitude ;;
  }

  dimension: longitude {
    hidden: yes
    type: number
    sql: ${TABLE}.longitude ;;
  }
}
```

---

## Task 4 — Group Similar Fields in Views

**What this solves:**  
Improves usability in the Explore UI by organizing related fields into logical groups for easier navigation.

### File: `views/users.view.lkml`

```lookml
dimension: age {
  group_label: "User Information (Challenge wF9m)"
  type: number
  sql: ${TABLE}.age ;;
}

dimension: city {
  group_label: "User Information (Challenge wF9m)"
  type: string
  sql: ${TABLE}.city ;;
}

dimension: country {
  group_label: "User Information (Challenge wF9m)"
  type: string
  sql: ${TABLE}.country ;;
}

dimension: state {
  group_label: "User Information (Challenge wF9m)"
  type: string
  sql: ${TABLE}.state ;;
}
```

---

### File: `views/products.view.lkml`

```lookml
dimension: brand {
  group_label: "Product Information (Challenge ONkO)"
  type: string
  sql: ${TABLE}.brand ;;
}

dimension: category {
  group_label: "Product Information (Challenge ONkO)"
  type: string
  sql: ${TABLE}.category ;;
}

dimension: department {
  group_label: "Product Information (Challenge ONkO)"
  type: string
  sql: ${TABLE}.department ;;
}

dimension: name {
  group_label: "Product Information (Challenge ONkO)"
  type: string
  sql: ${TABLE}.name ;;
}
```
