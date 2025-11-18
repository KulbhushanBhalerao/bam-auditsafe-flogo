
# Audit Safe - Business Activity Logging Testing Guide

> **Why Business Activity Monitoring?**  
> Learn more: [TIBCO Platform: Enabling Business Process Observability](https://www.tibco.com/blog/2025/08/16/tibco-platform-enabling-business-process-observability/)

## Table of Contents
- [Overview](#overview)
- [TIBCO Platform Testing](#tibco-platform-testing)
- [Local Testing](#local-testing)
   - [Prerequisites](#prerequisites)
   - [Configuration Steps](#configuration-steps)
- [API Reference](#api-reference)
   - [POST Event Endpoint](#post-event-endpoint)
   - [Request Example](#request-example)
   - [Response Example](#response-example)
- [Troubleshooting](#troubleshooting)
   - [Known Issues](#known-issues)

## Overview

This guide provides instructions for testing Business Activity Logging functionality on the TIBCO Platform, including both platform-based and local testing approaches.

## TIBCO Platform Testing

### Quick Start Guide

1. **Deploy** the Flogo application
2. **Expose** the endpoint externally
3. **Open** Swagger UI by clicking the Test button
4. **Invoke** the flow using sample messages
5. **View** Business Activity logging in the Business Activity Tab (located next to Logs)

## Local Testing

### Prerequisites

Set your namespace environment variable:

```bash
export DP_NS=<yourDPns>
```

### Configuration Steps

#### Step 1: Configure Post Events

Port forward the OTEL service:

```bash
kubectl port-forward svc/otel-userapp-logs -n $DP_NS 24318:24318
```

Set the endpoint environment variable:

```bash
export FLOGO_AUDITSAFE_OTLP_ENDPOINT=http://localhost:24318
```

#### Step 2: Configure Get Events

Port forward the O11y service:

```bash
kubectl port-forward svc/o11y-service -n $DP_NS 7820:7820
```

Set the endpoint environment variable:

```bash
export FLOGO_AUDITSAFE_O11Y_SVC_ENDPOINT=http://localhost:7820
```

#### Step 3: Run the Application

Execute the Flogo application in the same terminal where you configured the environment variables.

## API Reference

### POST Event Endpoint

**Endpoint URL:**

```
POST https://flogo.mle.atsnl-emea.azure.dataplanes.pro/tibco/apps/d4b01rlns2vs73fjfc10/ReceiveHTTPMessage/postevent
```

### Request Example

```bash
curl -X 'POST' \
   'https://flogo.mle.atsnl-emea.azure.dataplanes.pro/tibco/apps/d4b01rlns2vs73fjfc10/ReceiveHTTPMessage/postevent' \
   -H 'accept: application/json' \
   -H 'Content-Type: application/json' \
   -d '{
   "audit_event": "ORDER_DELIVERED",
   "business_process_name": "Order Management System",
   "event_description": "Order has been successfully delivered to the customer",
   "event_destination": "audit-safe-kafka-topic",
   "event_source": "order-service-api",
   "event_status": "COMPLETED",
   "event_timestamp": "2025-11-15T16:45:33.456Z",
   "payload": "{\"order_id\":\"ORD-20251113-5847\",\"customer_id\":\"CUST-98765\",\"customer_name\":\"John Smith\",\"order_date\":\"2025-11-13T14:32:15.123Z\",\"delivery_date\":\"2025-11-15T16:42:18.234Z\",\"total_amount\":1249.99,\"currency\":\"USD\",\"items\":[{\"product_id\":\"PROD-001\",\"product_name\":\"Laptop Computer\",\"quantity\":1,\"unit_price\":999.99},{\"product_id\":\"PROD-042\",\"product_name\":\"Wireless Mouse\",\"quantity\":2,\"unit_price\":125.00}],\"shipping_address\":{\"street\":\"123 Main Street\",\"city\":\"San Francisco\",\"state\":\"CA\",\"zip\":\"94102\",\"country\":\"USA\"},\"payment_method\":\"CREDIT_CARD\",\"payment_status\":\"CAPTURED\",\"delivery_status\":\"DELIVERED\",\"delivery_signature\":\"J.Smith\",\"carrier\":\"FedEx\",\"tracking_number\":\"FDX789456123\",\"delivered_by\":\"DRV-SF-028\",\"delivery_notes\":\"Left at front door as requested\"}",
   "transaction_id": "TXN-20251113-143218-A7B2C4"
}'
```

### Response Example

```
Audit entry created with tas_event_id: 455e8700-b755-41b7-ba8c-b3ed19d8236f, transaction_id: TXN-20251113-143218-A7B2C4 and event_id: 3757a3abff0d4ea6774436c1f2a0bb62
```
## Use Cases

### 1. BPM Flow Integration
In BPM Flows using Service activity which can call the REST APIs, you can call this flow to log the Business Activity.

### 2. Microservices Audit Trail
Track API calls and transactions across distributed microservices architectures to maintain a complete audit trail of business operations.

### 3. Order Processing Pipelines
Monitor and log key events in order fulfillment workflows, including order creation, payment processing, inventory updates, and delivery confirmation.

### 4. Compliance and Regulatory Reporting
Capture business events required for compliance with industry regulations (SOX, GDPR, HIPAA) by logging user actions, data access, and system modifications.

### 5. Customer Journey Tracking
Record customer interactions across multiple touchpoints to analyze behavior patterns and improve customer experience.

### 6. Financial Transaction Monitoring
Log financial transactions and accounting records for fraud detection, reconciliation, and audit purposes in banking and payment processing systems.

### 7. Supply Chain Events
Track goods movement, vendor interactions, and logistics events throughout the supply chain for visibility and accountability.

### 8. Error and Exception Handling
Capture system errors and business exceptions with contextual information for faster troubleshooting and root cause analysis.

### 9. SLA Monitoring
Record service request timestamps and completion status to measure and enforce Service Level Agreements.

### 10. Integration Flow Monitoring
Monitor data flow between integrated systems, capturing transformation events and data exchange activities.

### 11. Common Logging and Exception Handling
Centralize logging across all applications and services to create a unified view of system health, errors, and business events. Standardize exception handling by capturing error context, stack traces, and business impact for consistent troubleshooting and analysis.

### 12. Healthcare and Medical Records Management
Maintain comprehensive audit trails for nursing medical records, clinical research data, and health insurance information to ensure HIPAA compliance and patient data security.

### 13. E-Commerce Sales and Transaction Logging
Track e-commerce sales events, customer purchases, payment processing, and order fulfillment for business analytics and compliance.

### 14. Manufacturing Design Controls
Document and audit manufacturing design changes, version history, and approval workflows to maintain quality standards and regulatory compliance.

### 15. Content Management Version Control
Record document modifications, access events, and version changes in content management systems for accountability and change tracking.

### 16. IT Helpdesk and Support Records
Log helpdesk tickets, support requests, resolution steps, and user interactions for service quality monitoring and knowledge base development.

### 17. Legal and Research Investigations
Maintain tamper-proof audit logs of investigation activities, evidence handling, and research data access for legal proceedings and compliance.

### 18. Electoral and Voting Systems
Ensure transparent and auditable ballot-keeping and voting records to maintain election integrity and provide verifiable audit trails.

## Troubleshooting

### Known Issues

#### DP4 1.12 TP: Audit Safe Configuration from Global Observability

**Issue:**  
Audit safe configuration does not function correctly when linked from global observability.

**Workaround:**

1. **Configure DP Observability:**
    - Unlink from global observability
    - Copy configuration from global
    - Edit Business Activity log and export activity
    - Set index name to: `audit-safe`

2. **Fix Index Template (v1.12):**
    - Delete the existing index from Kibana
    - Restart the `o11y-service` pod
    - This recreates the index (e.g., `audit-safe-0001`) with correct mappings

3. **Verify:**  
    Invoke the service to confirm the fix is working


## Open questions
- UI or separate authorizations via TP CP for business users 
- IFrame support to embed into other UIs
- Elastic mangement, sharding, etc 