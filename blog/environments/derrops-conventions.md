---
slug: derrops-conventions
# title: Opensearch and Elasticsearch DSL is Better than SQL, you just don't know it yet
title: Derrops Guide to Naming Conventions and Segregation
date: 2026-02-26
authors: [derrops]
tags: [typescript, devops, aws]
draft: false
---


# Derrops Guide to Naming Conventions and Segregation

I've been meaning to make this guide, having worked at a SAAS from the very get-go, I've now got to live with some good and some bad decisions. It's been very hard to pinpoint why the naming conventions have lead to complexity. And overtime I realized that with engineering, you often make decisions based on the tradeoffs, but ultimately you do optimize for something. If you don't figure out what that something should be, you end having a solution which is not optimized for the problem you are trying to solve.

Living with a SAAS for many years now start to finish, what I ended up finding that these conventions are more important than I initially thought, and have ramifications in many different areas:

| Area           | Description                                                                                                      | Justification                                                                                                                                       |
|----------------|------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| *Cost*           | Ability to optimize, track, and control spend across environments, services, or resources.                       | Consistent naming and segregation make it easier to accurately allocate and monitor resource usage by environment, service, or team.                 |
| *Security*       | How easily you can enforce and audit security boundaries and practices.                                          | Segregation and clear naming provide obvious boundaries for applying Least Privilege, monitoring access, and investigating incidents.                |
| *Complexity*     | The overhead and cognitive load of understanding and managing environments and resources.                        | Predictable conventions reduce ambiguity and make it simpler for everyone to locate, manage, and reason about resources.                            |
| *Maintainability*| How straightforward it is to update, refactor, or scale your environments and naming conventions.                | Uniform conventions mean changes can be made systematically, minimizing the risk of errors and excessive rework.                                    |
| *Scalability*    | Ability to grow the number of services, accounts, or environments without hitting convention or collision issues.| Good conventions prevent collisions and manual workarounds as your organization and environments scale.                                              |
| *Usability*      | How easy naming and segregation is for engineers and consumers to work with daily.                               | Easy-to-understand resource names speed up onboarding and reduce misconfiguration by new and experienced engineers alike.                            |
| *Performance*    | Influence of structure/naming on ability to optimize for latency, throughput, or efficiency.                     | Logical segregation enables more effective resource grouping, which can positively impact locality, caching, or policy-based optimizations.           |
| *Reliability*    | Impact on operational stability, incident response, and minimizing blast radius of issues.                       | Separation by environment/service/domain helps contain failures and allows more targeted incident response.                                           |
| *Availability*   | Affect on uptime or redundancy due to naming/segregation conventions.                                            | Clear segregation facilitates independent deployments and failover, increasing uptime and resilience.                                                 |
| *Compliance*     | Helps enforce regulatory, audit, and governance requirements more easily.                                        | Accurate, well-segregated naming enables easier reporting, automated policy enforcement, and audit compliance.                                        |


These areas can all be condensed into the following guiding principals


# Guiding Principles

## 1. Naming Consistency Principle

**Definition:**  
A given resource should have the *same name* in every environment—whether it’s `dev`, `prod`, or any other. For example, a resource called `user-service` should be named `user-service` in all environments, rather than `user-service-dev` or `user-service-prod`. Environment must be represented by namespace isolation, not naming convention.

**Why it matters:**  
Consistent resource names dramatically simplify the process of grouping, querying, and managing deployment instances across various tools and platforms. It helps reduce cognitive load and operational complexity, especially as your infrastructure grows.

:::note
Globally unique name, such as with an s3 bucket, it can be that another customer on AWS takes the name of an S3 bucket that you had planned. One solution to this is including a random id within the name. But in general I've found this scenario rare, and losing predictability in the name which can lead to negative outcomes
:::

:::note
Global services such as IAM which are global also may cause conflicts if you were to have more than 1 environment in an account **Prefer Account per Environment if Possible**
:::


---

## 2. Naming Stability Principle

**Definition:**  
Prioritize naming conventions that minimize how often resource names need to change over the lifetime of your systems. Choose prefixes or structures for your names that are unlikely to require revision as your organization and requirements evolve.

**Why it matters:**  
Stable naming reduces refactoring, lowers risk of errors, and leads to smoother automation and scaling. The fewer changes needed, the more reliable and maintainable your environment will be in the long run.
You'll also find that following this principal results in a better security posture, more on that later.








# Segregation Strategy

Segregation needs to be done in a strategic way. Resources segregated differently can lead to drastically different outcomes. Following a consistent pattern for `Segregation and Naming Conventions` can lead ot many different benefits:
 - To avoid naming collisions
 - To reduce complexity by not adding unnecessary naming conventions to resources.
 - To make it easy to group resources across environments together. This is useful in filtering duplicates for security findings, as having different names for the same resources adds complexity to the queries needed 
 - Easier to automate as resources are easier to target across different environments as they are named in a predictable way.
 - Within an environment, segregating services and organizations sharing an environment increases your security posture, as the prefixes can be used to scope access to resources and create access boundaries

## Account Segregation

How you segregate your account will determine your name space, and therefore will affect your naming conventions. When creating accounts, modern patters would be to have 1 per region and environment, but there may be cases where you take a different approach again because of other factors related to: (*Cost&, *Compliance*, *Security*, etc).
But do challenge this decision if you do not segregate, as there is now especially strong multi-account support in Cloud Providers such as AWS.

> In general the more you segregate, the less likely you'll have naming collisions.

| Segment | Name | Description |
|---------|------|-------------|
| Region | `{region}` | If you don't segregate your Account by region, you will need to have `{region}` in the name of the resource to avoid collisions. |
| Environment | `{env}` | If you don't segregate your Account by environment, you will need to have `{env}` in the name of the resource to avoid collisions. |


:::tip
Operationally if `Deployment Instances` have different names, it can create severe complexity when trying to groups those instances together in different DevOps tools and products. I've experienced this first hand when sifting through findings in AWS Security Hub. Just when you find a way within a given tool to group these resources and solve this, as your organization grows and adopts new tools you find yourself being faced with this issue again and again for every tool you use.
:::

:::note
Consider comparing 2 x Lambda functions `foobar`, in 2 different environments `dev` and `prod`.
If the Deployment Instances have different names, your query will need to have an OR clause to include both, `foobar-dev` or `foobar-prod` and then also have the group by environment anyway. Whereas if they have the same name, there is no need for a clause on the name. Excluding environment reduces the mental load in many different scenarios.
:::


## Prefixing

### Segregation Hierarchy    

```mermaid
graph LR
    R[Region]
    E[Environment]
    O[Organization]
    D[Domain]
    S[Service]
    R --> E
    E --> O
    O --> D
    D --> S
```


### Prefix Structure

Structuring your names correctly is key to the success of your naming conventions.

If your config store is **NOT** located in the same namespace as the resource, you will have to have `{env}` in your naming conventions. It's challenging where to put `{env}`, as there are tradeoffs
Also note if your config store is **NOT** in the same `{region}`, then you will need to have `{region}` in your naming conventions.




## Segment Definitions

| Segment | Example            | Description                                                                                                                             |
|---------|--------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| `{region}`     | ap-southeast-2               | *(Optional if platform will be multi-region)* The region of the deployment. This may be required if you are not going to segregate your regions by accounts, then there will be conflicts as some services are global (AWS IAM) or share the same namespace (such as S3 Buckets which must be unique across all regions). Deciding to have different accounts for different regions mitigates this issue, but this decision can sometimes be taking other factors into account than this guide. Note this is the Datacenter Region (e.g. ap-southeast-2), not the country itself. | 
| `{env}`   | dev                | *(Only if Required)* The deployment environment. Only required in the prefix, when the config key lives in a different namespace than the resource accessing it. If your config is stored in the same account or namespace as the resource, this segment is redundant and should be omitted. However if a globally unique name is required, such as with an s3 bucket, this this will be required in the prefix.|
| `{org}`     | acme               | The top-level tenant or business unit. Provides hard namespace isolation. Chosen to be stable and long-lived—changes are rare and treated as a migration event. |
| `{domain}`  | payments           | A bounded business or technical capability that can be owned and reasoned about independently. More stable than a team name, more meaningful than a platform label. |
| `{service}` | checkout-api       | The concrete, deployable unit within a domain. Specific enough to be unambiguous, broad enough to own multiple configuration values beneath it. |
| `{key}`     | stripe-webhook-secret | The actual parameter being addressed. Everything above it is context and namespace; this is the value you are looking up.                     |

 
## Segment Order

It is sometimes said:

> Good architecture makes change easy. Bad architecture makes change hard.

Therefore in this decision we should optimize for making `Change Easy`, which would dictate the 

> stability should decrease left to right in a prefix

- Changing leftmost segments causes the greatest disruption.
- Renaming service only affects its subtree.

Therefore it makes sense the order of the segments should be the most stable to the least stable:

## Segment Stability

| Segment | Question | Stability | Scope Boundary | Stability |
|---------|----------|-----------|----------------|-----------|
| `{region}`  | **Where** in the world? | Extremely High | Infrastructure locality boundary | for all intensive purposes will never change |
| `{env}`  | **Which** deployment stage? | Extremely High | Deployment lifecycle boundary | for all intensive purposes will never change |
| `{org}`  | **Who** owns the resource? | Very High | Ownership boundary | changes almost never, unless re-org |
| `{domain}` | **What** business capability? | High | Capability boundary | changes rarely, capabilities outlive teams, only if domain is modelled differently and refactored |
| `{service}` | **Which** deployable unit? | Medium | Deployment unit boundary | changes occasionally, deployable units get renamed, split or merged |
| `{key}` | **What** configuration value? | Low | Configuration/value boundary | changes most frequently |

### Fully Qualified Examples

| Example | Value |
|---------|-------|
| Hierarchical (config stores) | `/ap-southeast-2/prod/acme/payments/checkout-api/stripe-webhook-secret`
| Compound kebab-case (resource names) | `/ap-southeast-2/prod/acme/payments/checkout-api/stripe-webhook-secret`
| Account-segregated (preferred): | `acme--payments--checkout-api--stripe-webhook-secret` |


### Delimiters

Now that we have segments, and their order, we need to decide on the delimiters to use.
Here are some possible candidates evaluated over different resource types.
You'll see that the `-` is most supported. `_` is a strong candidate but fails when it comes to host names and S3 bucket names.

**Possible Candidates for Delimiters:**
| Delimiter          | Hostname | AWS Stack Name (CloudFormation) | URL Path      | S3 Bucket Name               | Safe Across ALL? | Notes                                               |
| ------------------ | -------- | ------------------------------- | ------------- | ---------------------------- | ---------------- | --------------------------------------------------- |
| `-` hyphen         | ✅ Yes    | ✅ Yes                           | ✅ Yes         | ✅ Yes                        | ⭐ Yes (BEST)     | Universal standard. Recommended everywhere.         |
| `_` underscore     | ❌ No     | ✅ Yes                           | ✅ Yes         | ❌ No                         | ❌ No             | Breaks hostname and S3 bucket compatibility         |
| `.` dot            | ✅ Yes    | ✅ Yes                           | ✅ Yes         | ✅ Yes (with caveats)         | ⚠️ Conditional   | Breaks TLS wildcard matching, avoid in bucket names |
| `/` slash          | ❌ No     | ❌ No                            | ✅ Yes         | ❌ No (delimiter only in key) | ❌ No             | Only valid as URL path separator                    |
| space              | ❌ No     | ❌ No                            | ❌ Encoded     | ❌ No                         | ❌ No             | Never use                                           |
| `--` double hyphen | ✅ Yes    | ✅ Yes                           | ✅ Yes         | ✅ Yes                        | ⭐ Yes            | Common and safe variant                             |


There are different types of delimiters. Sometimes we are representing differences between the different segments of the prefix, other times we are representing differences between the different words with the same segment.
Therefore we need a delimiter for each case.
As `-` is the only supported delimiter for all resource types, including another is not ideal because it will reduce the resources types which can follow that convention. Instead the `--` is used to delimit between the different segments of the prefix, and a `-` is used to delimit between the different words with the same segment.

| Naming Convention      | Priority  | Resource Type   | Delimiter | Word Delimiter | Description                                                                                                                         | Example                                |
|------------------------|-----------|-----------------|-----------|----------------|-------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------|
| *hierarchical-case*    | 1         | Config Keys     | `/`       | `-`            | Hierarchical naming convention using a path-style namespace, useful for configuration keys.                                         | `/acme/store/checkout/api-key`         |
| *compound kebab-case*  | 2         | Resource IDs    | `--`      | `-`            | Kebab-case is URL and code friendly; more acceptable than underscores for most services, especially for resource IDs.                | `acme--store--checkout--api-feature`   |

- Often `1` is not possible as a name (for example IAC solutions such as AWS Cloudformation)
- `-` is preferred over `_` as it is more URL friendly (can appear in the host name where as underscore cannot).
- `--` whilst this doesn't look the most pleasing to the eye, it is more URL friendly and distinguishes between the hierarchial delimiters and word delimiters.
- other tools will not use `--` as a delimiter, so using it as a delimiter in the prefix for a key will not be compatible with other tools.



### Native Delimiters
:::note
If a resource already has native support for a delimiter, it should be used instead of the `--` delimiter, such as `/` in AWS S3, for storing objects. Otherwise you would lose functionality such as using the `prefix` to filter objects in a bucket.
:::

:::note
An exception to this rule might be when you plan on centralizing data collection, and you want all data to be stored in a single location. You may want to preemptively segregate the data in each environment already, even though there will be only 1 segment, to simplify the sync operation.
:::

:::note
We will assume going forward there is only 1 region, so there is no need for segregation by region, but if there wasn't, region would need to be included in the prefix
:::

**Full example:**
 1. `/{org}/{domain}/{service}/{key}` => `/acme/payments/checkout-api/stripe-webhook-secret`
 2. `{org}--{domain}--{service}--{key}` => `acme--payments--checkout-api--stripe-webhook-secret`


## Rational for Convention

#### Every prefix is a meaningful operational boundary 

| Prefix | Meaning |
|--------|---------|
| `/acme/*` | Everything in the org | 
| `/acme/payments/*` | Everything owned by payments | 
| `/acme/payments/checkout-api/*` | Everything for a specific service |
| `/acme/payments/checkout-api/stripe-webhook-secret` | A specific value |

If `{env}` and `{region}` would be in there then 


| Prefix | Meaning |
|--------|---------|
| `/ap-southeast-2/*` | Everything in the region | 
| `/ap-southeast-2/prod/*` | Everything in the environment | 
| `/ap-southeast-2/prod/acme` | Everything in the org, in that env | 


## Segment Definitions

### Region
The region identifies the physical or logical infrastructure region where the deployment instance resides.

Examples:
 - ap-southeast-2
 - us-east-1
 - eu-west-1

### Org
The org identifies the top-level organizational boundary. Which department owns the resource.
:::note
Not the team itself as teams can change more frequently than the org itself, otherwise this becomes too unstable to use as a namespace boundary.
:::

### Domain

- Encodes business capability rather than org structure — capabilities are far more stable than teams or products over time
- Well understood in modern engineering culture thanks to DDD, Team Topologies, and microservices discourse
- Naturally guides correct usage — engineers intuitively know whether something belongs to payments or identity
- Aligns with how most mature orgs already think about their architecture, even if they don't use the word

### Service

The service represents the concrete deployable unit. This is the actual runtime component which is typically the primary identity engineers interact with..

Examples:
 - checkout-api
 - auth-service
 - webhook-worker
 - billing-scheduler

Deployment Units could be in the form of:
 - Lambda function
 - Microservice

## Key
The key identifies a specific configuration value, secret, resource, or parameter belonging to a service.
Represents the exact value being referenced.
Everything to the left provides context.

 - Created
 - Deleted
 - Renamed
 - Rotated


:::note
Optional: This principal, when dogmatically applied may mean that you segregate data in an environment, even when there will only be 1 segment, to simplify the sync operation to a central location later.
As the end destination will need to be segmented, but the source data not necessarily.
But by segregating the data in the source we achieve this principle.

This contradicts another principle in guiding how to segregate data, which would say if you only were going to have 1 segment, you should not segregate the data in the first place. But if you can see the future, and you know that you are going to combine data together, then you are best off already pre-paring for this scenario so it doesn't hit you later.
:::



## Tagging (Suggestions)

### Tagging Env
Many would argue that you need to tag every resource with what environment it lives in. I would argue that if you are segregating environments by accounts, then tags are redundant. Tagging resources would then violate the DRY Do Not Repeat Yourself principle, and add to overhead in needing more unnecessary tagging policies. Whilst it's fairly easy to tag resources you manage, sometimes tools or other entities manage resources in your account and it can be problematic to tag them.

`Account` itself is a stronger way (than tagging) to group resources across an environment together, as an `Account` is an actual **container** for resources, and not just a **label**. It's easier to make mistakes and not tag a resource for some amount of time, cause cost allocation calculations to be incorrect, whereas usage by `Account` is always accurate.


If there is some other need to tag resources, such as for security or compliance or because of a tool or service you are using.
Then have the `Account` the source of truth and perform batch tagging as needed. This will reduce the *Maintainability*.




# Concrete Examples:

For the naming convention we don't want to have the `{env}` and `{region}` because of the `Naming Consistency Principle`.
But because of uniqueness constraints within a namespace this is not always possible. See uniqueness constraints below:


### Uniqueness Constraints

| Service | Scope Boundary | region | env | Example |
|---------|----------------|--------|-----|---------|
| Global Namespace | Globally Unique across all Namespaces | ✅ | ✅ | AWS S3 Bucket |
| Global Service | Unique in all Regions within a Namespace | ✅ | ❌ | IAM Role Name |
| Regional Namespace |Unique in a Region within a Scope Boundary | ❌ | ❌ | RDS Instance Identifier |


### Concrete Example


| Resource Type                 | Scope      | Global? | Physical Name                                                      | `{region}` required | `{env}` required | Rationale               |
| ----------------------------- | ---------- | ------- | ------------------------------------------------------------------ | ------------------- | ---------------- | ----------------------- |
| S3 Bucket                     | Global     | ✅       | ap-southeast-2--prod--acme--payments--checkout-api--backup-storage | ✅                   | ✅                | Must be globally unique |
| Route53 Hosted Zone           | Global     | ✅       | prod.acme.com                                                      | ❌                   | ✅                | Each env account is delegated its own subdomain of the apex domain |
| Route53 Record                | Zone scope | ✅       | checkout-api.prod.acme.com                                         | ❌                   | ✅                | DNS global namespace    |
| CloudFront Distribution Alias | Global     | ✅       | checkout-api.prod.acme.com                                         | ❌                   | ✅                | Global DNS namespace    |
| ACM Certificate Domain        | Global     | ✅       | checkout-api.prod.acme.com                                         | ❌                   | ✅                | Global DNS namespace    |
| S3 Static Website             | Global     | ✅       | prod.acme-checkout-api                                             | ✅                   | ✅                | Global namespace        |



# Domain and DNS Naming Conventions

DNS naming is a special case where hierarchy is reversed compared to prefix-based naming conventions.

Prefix-based systems grow **left → right**:

```
/{org}/{domain}/{service}
```

DNS grows **right → left**:

```
{service}.{domain}.{org}.com
```

Both represent the same logical hierarchy in opposite directions.

---

## Core Principle

> DNS names must preserve the same logical hierarchy as prefixes, but in reverse order.

| Prefix                        | DNS Equivalent                   |
| ----------------------------- | -------------------------------- |
| `/acme/payments/checkout-api` | `checkout-api.payments.acme.com` |
| `/acme/identity/auth-service` | `auth-service.identity.acme.com` |
| `/acme/portal/web`            | `web.portal.acme.com`            |

Hierarchy equivalence:

| Logical Level | Prefix       | DNS                            |
| ------------- | ------------ | ------------------------------ |
| Org           | acme         | acme.com                       |
| Domain        | payments     | payments.acme.com              |
| Service       | checkout-api | checkout-api.payments.acme.com |

---

## Environment Inclusion

Environment appears in DNS between the service and the org, following the convention:

```
{service}.{env}.{org}.com
```

Each environment account is delegated its own subdomain of the apex domain:

```
checkout-api.prod.acme.com   (prod account owns prod.acme.com)
checkout-api.dev.acme.com    (dev account owns dev.acme.com)
```

The apex domain (`acme.com`) is managed in a central network or DNS account. Each environment account is delegated a subdomain (`prod.acme.com`, `dev.acme.com`), giving that account full ownership and autonomy over its DNS records. This mirrors how account segregation provides namespace isolation — the subdomain **is** the account's namespace in DNS.

This approach has a key advantage: any new service deployed into an account is automatically under that account's subdomain, with no coordination required at the apex level.

Alternative (apex zone per account, no env in DNS):

```
checkout-api.payments.acme.com
```

Same name exists independently in each environment account. Env isolation is provided entirely by the account boundary with no qualifier in the URL.

---

## Route53 Hosted Zone Strategy

Preferred:

The apex domain (`acme.com`) is managed in a central account. Each environment account is delegated its own subdomain via NS record delegation:

```
prod.acme.com   (prod account)
dev.acme.com    (dev account)
uat.acme.com    (uat account)
```

Services are then addressed as `{service}.{env}.{org}.com`:

```
checkout-api.prod.acme.com
```

Each account has full autonomy over its subdomain. Adding a new service or record requires no changes to the central apex zone. The apex zone only needs to be updated when a new environment account is onboarded.

Alternative (apex zone per account):

```
acme.com
```

Environment isolation is provided entirely by the account boundary. No env qualifier appears in DNS.

---

## Summary

| Prefix Convention            | DNS Convention                  |
| ---------------------------- | ------------------------------- |
| `{org}/{domain}/{service}`   | `{service}.{env}.{org}.com`     |
| Hierarchy grows left → right | Hierarchy grows right → left    |
| Namespace via account        | Namespace via env subdomain     |
| Logical identity preserved   | Logical identity preserved      |

DNS is not an exception to the naming convention — it is the same hierarchy represented in reverse due to DNS delegation design.


**Cross cutting secrets/config**
:::info
Sometimes there will be cross cutting secrets/config which do not belong to any one particular domain, service or organization. Such as if the organization has purchased a service to be used by 2 different orgs. If this configuration cannot be definitively assigned to one or the other, then it should be stored in a cross cutting location, and permissions will need to be explicitly granted, rather than relying on prefix conventions.
:::


**Logical Name vs Physical Name**

Logical Name:
checkout-api

Physical Deployment Instance:
Account: prod
Region: ap-southeast-2
Name: checkout-api

Fully Qualified Identity:
ap-southeast-2/prod/checkout-api

Name remains constant. Namespace varies.

**Deployment Instance**

A Deployment Instance is a concrete runtime instantiation of a logical resource within a specific namespace.

Example:

Logical Resource: checkout-api

Deployment Instances:
- Account: dev → checkout-api
- Account: prod → checkout-api
- Account: uat → checkout-api

All share the same logical name but exist in different namespaces.