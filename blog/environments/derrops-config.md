---
slug: derrops-config
# title: Opensearch and Elasticsearch DSL is Better than SQL, you just don't know it yet
title: Derrops Guide to Config
date: 2026-02-24
authors: [derrops]
tags: [typescript, devops, aws, config]
draft: true
---


# Configuration & Secrets Ownership Guide

This document defines **where configuration and secrets live**, **why**,
and **the rules governing each layer**.\
The goal is to make configuration **explicit, auditable, secure, and
predictable**.

------------------------------------------------------------------------

## Where does configuration live?

Configuration is split into **layers**, each with a **clear
responsibility**.\
No single system should answer every question.


# Configuration Authority Hierarchy

Each configuration value has exactly one authority.
| Authority                        | System / Layer                     | Responsibility             | Example                        |
|----------------------------------|------------------------------------|----------------------------|---------------------------------|
| **Local Authority**              | Env File (.env file)               | Local dev enablement       | Allowed regions, schemas        |
| **Policy Authority**              | Git (repository)                  | What is allowed            | Allowed regions, schemas        |
| **Runtime Authority**             | SSM Parameter Store               | Defines what is active     | Feature flags, live endpoints   |
| **Secret Authority**              | Secrets Manager                   | Defines secret values      | API keys, passwords             |
| **Sensitive Config Authority**    | SecureString                      | Manages sensitive config   | Encrypted DB URI                |
| **Validation Authority**          | Code schema validation            | Ensures correctness        | Zod/Valibot/Yup schema checks   |
| **Consumption Authority**         | Application code                  | Consumes config            | Reads config object in code     |



------------------------------------------------------------------------

## .env files (Local Development Layer)

`.env` files are used only for local development.

## Purpose

-   Provide secrets and configuration for local execution
-   Allow developers to run services without cloud dependency

## Rules

-   Must not be committed to Git
-   Must be gitignored
-   Must only exist locally
-   Must not be used in production
-   Must follow the same schema as production configuration

------------------------------------------------------------------------

# Git (Policy Layer)

**Git is the source of truth for intent and policy.**

It answers *why* something exists, not *what it currently is*.

## Git defines

-   Allowed values
-   Schemas
-   Constraints
-   Policy-level defaults
-   Environment shapes

## Git must not contain

-   Secrets
-   Runtime values
-   Environment-specific endpoints
-   Credentials

## Rules

-   All changes require review
-   All changes must be auditable
-   Git defines policy, not runtime state

------------------------------------------------------------------------

# SSM Parameter Store (Runtime Configuration Layer)

**SSM represents runtime configuration state.**

## Rules

-   Values must conform to Git policy
-   Values must be environment-scoped
-   Changes must not require rebuilds
-   Parameters must be hierarchical and namespaced
-   Secrets Manager should be preferred over SSM for secrets

## Naming Convention

    `/{org}/{system}/{env}/{service}/{key}`

Example:

    /fortiro/protect/prod/api/feature/scan-enabled
    /fortiro/protect/prod/api/db/host

------------------------------------------------------------------------

# Secrets Manager (Primary Secrets Layer)

**Secrets Manager is the preferred system for managing secrets.**

## Rules

-   Must be injected at runtime
-   Must never be hardcoded
-   Must never be committed to Git
-   Must be readable only by authorised services
-   Rotation should be enabled when possible

------------------------------------------------------------------------

# Tags (Metadata Layer)

Tags define ownership and metadata.

## Rules

-   All AWS resources must be tagged
-   Tags must never control application behavior
-   Tags must not contain secrets

------------------------------------------------------------------------

# Codebase (Consumer Layer)

The codebase consumes configuration.

## Rules

-   Must not define runtime values
-   Must not define secrets
-   Must validate schema
-   Must fail fast on missing configuration

------------------------------------------------------------------------

# Fail Fast Requirement

Application startup must fail if:

-   Required configuration is missing
-   Secrets cannot be retrieved
-   Configuration violates schema

------------------------------------------------------------------------

# Configuration Injection Model

Configuration must be injected at runtime via:

-   Environment variables
-   SSM Parameter Store
-   Secrets Manager

Configuration must never be compiled into the application artifact.

------------------------------------------------------------------------

# Runtime Defaults Policy

Defaults must exist only in configuration, not application logic.

Allowed:

    size: params.size ?? config['app.pagination.default.size']

Forbidden:

    size: params.size ?? 20

------------------------------------------------------------------------

# Environment Precedence

Highest precedence first:

1.  Explicit test configuration injection
2.  System environment variables
3.  Local `.env`
4.  SSM Parameter Store
5.  Secrets Manager
6.  Git policy
7.  Code schema validation

------------------------------------------------------------------------

# Result

This model ensures:

-   Secure secret handling
-   Explicit runtime configuration
-   Fail-fast startup guarantees
-   Predictable deployments


# Glossary

 - Deployment Instances