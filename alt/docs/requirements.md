# Bed Track: Real-time Bed Availability Tracking

**Architecture & Deployment Requirements**

Owner: Platform Engineering
Last updated: 2026-09-06

## Overview

BedTrack is the real-time bed availability tracking service for the hospital platform. Ward systems publish bed status events (occupied, available, cleaning, maintenance) which are processed, stored, and used to alert care coordinators when beds become available in high-demand units like ICU and ED overflow.

This is a PHI-scope service running on AWS. All data must be encrypted at rest and in transit using a Customer-Managed KMS Key.

## Deployment

- **Cloud:** AWS
- **Region:** us-west-2 — co-located with the primary hospital network. PHI must stay in this region; do not deploy elsewhere without CISO sign-off.
- **Environment:** Production
- **Multi-AZ:** Yes — required across compute and data tiers.

## Architecture

Ward systems → SQS queue → Lambda processor → DynamoDB (real-time state) + SNS (alerts) + S3 (audit archive)

Failed events (after 3 retries) go to a DLQ for ops review.

## Components

- **Lambda** — Python 3.12, runs in private VPC subnets, consumes SQS in batches with partial failure support. X-Ray tracing on.
- **SQS** — Main queue feeds Lambda. DLQ retains bad messages for 14 days for ops review, per Ops' request for a longer investigation window on failed ward-event batches. Both CMK-encrypted.
- **DynamoDB** — Real-time bed state. PAY_PER_REQUEST, CMK encrypted, PITR enabled. GSI on `ward_id` for per-ward queries. 90-day TTL; long-term data lives in S3.
- **SNS** — Alerts care coordinators when a bed flips to available in a flagged unit. CMK encrypted, email subscription.
- **S3** — HIPAA audit archive. Versioned, CMK encrypted, no public access. Lifecycle transitions to Glacier at 1 year, expires at 6 years (HIPAA minimum).
- **KMS** — Single CMK (`alias/bedtrack-phi-cmk`) used across all data stores. Annual rotation, minimum 7-day deletion window.

## IAM

Lambda execution role should be least-privilege — scoped to the specific queues, table, topic, and bucket prefix it needs. No wildcards on data resources.

## Tagging

All resources must be tagged: `app=bedtrack`, `env=production`, `data-sensitivity=phi`, `hipaa-scope=true`.

## Open Items

- Confirm care coordinator alert email for SNS subscription
- Network Engineering to provision us-west-2 VPC, subnets, and Lambda SG before first deploy
