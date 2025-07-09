# OpenSearch Monitoring with Prometheus Recording Rules

This repository contains monitoring configuration for OpenSearch using Prometheus and Grafana, with a focus on recording rules for efficient metrics collection and alerting.

## What are Recording Rules?

Recording rules in Prometheus are pre-computed metrics that are evaluated at regular intervals and stored as time series data. They serve several important purposes:

### Key Benefits:

1. **Performance Optimization**: Instead of running complex queries repeatedly, recording rules pre-compute results and store them as new metrics
2. **Reduced Query Latency**: Dashboards and alerts can query pre-computed metrics instead of calculating them on-demand
3. **Consistency**: All users see the same computed values since they're calculated once and stored
4. **Resource Efficiency**: Reduces CPU load on Prometheus server by avoiding repeated expensive calculations
5. **Alerting Foundation**: Provides stable, pre-computed metrics for reliable alerting

## Recording Rules in This Repository

The `distribution/prometheus_recording_rules.yaml` file contains OpenSearch-specific recording rules:

### Bulk Operations Monitoring

- **`bulk:rejected_requests:rate2m`**: Calculates the 2-minute rate of rejected bulk requests
  - Source: `rate(opensearch_threadpool_threads_count{name="bulk", type="rejected"}[2m])`
  - Use: Monitoring bulk request rejection trends

- **`bulk:completed_requests:rate2m`**: Calculates the 2-minute rate of completed bulk requests
  - Source: `rate(opensearch_threadpool_threads_count{name="bulk", type="completed"}[2m])`
  - Use: Monitoring bulk request completion trends

- **`bulk:reject_ratio:rate2m`**: Calculates the ratio of rejected to completed bulk requests
  - Source: `sum by (cluster, instance, node) (bulk:rejected_requests:rate2m) / on (cluster, instance, node) (bulk:completed_requests:rate2m)`
  - Use: Identifying nodes with high rejection rates

### Integration with Alerting

These recording rules are used by the alerting system in `distribution/prometheus_alerts.yaml`:

- **OpenSearchBulkRequestsRejectionJumps**: Triggers when `bulk:reject_ratio:rate2m` exceeds 5%
- Provides stable, pre-computed metrics for consistent alerting behavior

### Dashboard Integration

The Grafana dashboard (`distribution/opensearch.json`) can utilize these recording rules for:
- Real-time monitoring of bulk operation health
- Historical trend analysis with consistent metrics
- Reduced dashboard loading times

## Usage

1. Deploy the recording rules to your Prometheus server
2. Configure the alert rules to use the pre-computed metrics
3. Import the Grafana dashboard for visualization
4. Monitor OpenSearch bulk operations with improved performance and consistency

## File Structure

```
distribution/
├── prometheus_recording_rules.yaml  # Pre-computed metrics definitions
├── prometheus_alerts.yaml          # Alert rules using recording rules
└── opensearch.json                 # Grafana dashboard configuration
```
