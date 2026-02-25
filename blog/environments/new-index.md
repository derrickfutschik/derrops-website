---
slug: new-environments
# title: Opensearch and Elasticsearch DSL is Better than SQL, you just don't know it yet
title: Derrops Conventions
date: 2026-02-24
authors: [derrops]
tags: [typescript, devops, aws]
draft: true
---



# Segregation and Naming Conventions

I've been meaning to make this guide, having worked at a SAAS from the very get-go, I've now got to live with some good and some bad decisions.

## Naming Resources

:::tip
Strongly Recommend Naming Resources should be Agnostic of Environment
:::

Now especially that there are strong namespaces capabilities in Cloud Providers such as AWS, as well as strong support for multi-account setups, I recommend **NOT** including environment names in resource names, or even tags for that matter. An `Account` itself is a stronger way (than tagging) to group resources across an environment together, as an `Account` is an actual **container** for resources, and not just a **label**.

Operationally if `Deployment Instances` have different names, it can create severe complexity when trying to groups those instances together in different DevOps tools and products. I've experienced this first hand when sifting through findings in AWS Security Hub. Just when you find a way within a given tool to group these resources and solve this, as your organization grows and adopts new tools you find yourself being faced with this issue again and again for every tool you use.

:::note
Consider comparing 2 x Lambda functions `foobar`, in 2 different environments `dev` and `prod`.
If the Deployment Instances have different names, your query will need to have an OR clause to include both, `foobar-dev` or `foobar-prod` and then also have the group by environment anyway. Whereas if they have the same name, there is no need for a clause on the name. Excluding environment reduces the mental load in many different scenarios.
:::


## Segregation Strategy

Segregation needs to be done in a strategic way. Resources segregated differently can lead to drastically different outcomes. Following a consistent pattern for `Segregation and Naming Conventions` can lead ot many different benefits:
 - To avoid naming collisions
 - To reduce complexity by not adding unnecessary naming conventions to resources.
 - To make it easy to group resources across environments together. This is useful in filtering duplicates for security findings, as having different names for the same resources adds complexity to the queries needed 
 - Easier to automate as resources are easier to target across different environments as they are named in a predictable way.
 - Within an environment, segregating services and organizations sharing an environment increases your security posture, as the prefixes can be used to scope access to resources and create access boundaries


## Segregation Hierarchy    

```mermaid
graph LR
    E[Environment]
    O[Organization]
    D[Domain]
    S[Service]
    E --> O
    O --> D
    D --> S
```

## Key Naming Conventions

### Delimiters

| Naming Convention      | Priority  | Resource Type   | Delimiter | Word Delimiter | Description                                                                                                                         | Example                                |
|------------------------|-----------|-----------------|-----------|----------------|-------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------|
| *hierarchical-case*    | 1         | Config Keys     | `/`       | `-`            | Hierarchical naming convention using a path-style namespace, useful for configuration keys.                                         | `/acme/store/checkout/api-key`         |
| *compound kebab-case*  | 2         | Resource IDs    | `--`      | `-`            | Used by AWS internally. Kebab-case is URL and code friendly; more acceptable than underscores for most services, especially for resource IDs.                | `acme--store--checkout--api-feature`   |

- Often `1` is not possible as a name (for example IAC solutions such as AWS Cloudformation)
- `-` is preferred over `_` as it is more URL friendly (can appear in the host name where as underscore cannot).
- `--` whilst this doesn't look the most pleasing to the eye, it is more URL friendly and distinguishes between the hierarchial delimiters and the word delimiters.
- other tools will not use `--` as a delimiter, so using it as a delimiter in the prefix for a key will not be compatible with other tools.


### Prefix Structure

If your configuration is **NOT** stored in the same namespace as the resource, you will have to have `{env}` in your naming conventions. It's challenging where to put `{env}`, as there are tradeoffs

| Segment | Example            | Description                                                                                                                             |
|---------|--------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| `{env}`   | dev                | *(Only if Required)* The deployment environment. Only required when the config key lives in a different namespace than the resource accessing it. If your config is stored in the same account or namespace as the resource, this segment is redundant and should be omitted. |
| `{org}`     | acme               | The top-level tenant or business unit. Provides hard namespace isolation. Chosen to be stable and long-lived—changes are rare and treated as a migration event. |
| `{domain}`  | payments           | A bounded business or technical capability that can be owned and reasoned about independently. More stable than a team name, more meaningful than a platform label. |
| `{service}` | checkout-api       | The concrete, deployable unit within a domain. Specific enough to be unambiguous, broad enough to own multiple configuration values beneath it. |
| `{key}`     | stripe-webhook-secret | The actual parameter being addressed. Everything above it is context and namespace; this is the value you are looking up.                     |

**Full example:**
 1. `/{org}/{domain}/{service}/{key}` => `/acme/payments/checkout-api/stripe-webhook-secret`
 2. `{org}--{domain}--{service}--{key}` => `acme--payments--checkout-api--stripe-webhook-secret`


# TODO Edge Cases

 1. Cross cutting secrets/config which do not belong to any one particular domain, service or organization.

## Rational for Convention

#### Every prefix is a meaningful operational boundary 

| Prefix | Meaning |
|--------|---------|
| `/acme/*` | Everything in the org | 
| `/acme/payments/*` | Everything owned by payments | 
| `/acme/payments/checkout-api/*` | Everything for a specific service |
| `/acme/payments/checkout-api/stripe-webhook-secret` | A specific value |

### Stability increases left to right
Each segment is chosen to be more volatile than the one to its left:

| Segment | Stability |
|---------|-----------|
| `{org}`  | changes almost never |
| `{domain}` | changes rarely, capabilities outlive teams |
| `{service}` | changes occasionally, deployable units get renamed or split |
| `{key}` | changes most frequently |


## Each Component

### Org - Organizational Segregation

### Domain

- Encodes business capability rather than org structure — capabilities are far more stable than teams or products over time
- Well understood in modern engineering culture thanks to DDD, Team Topologies, and microservices discourse
- Naturally guides correct usage — engineers intuitively know whether something belongs to payments or identity
- Aligns with how most mature orgs already think about their architecture, even if they don't use the word



to reduce operational complexity.





### Service

## Environment Segregation

 - Segregate environments by account so that naming collisions are avoided, whilst reducing complexity by not adding unnecessary naming conventions to resources.
 - By having the same name for a resource in different environments, it makes it easy to group resources across environments together. This is useful in filtering duplicates for security findings, as having different names for the same resources adds complexity to the queries needed 



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