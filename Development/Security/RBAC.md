---
aliases:
  - RBAC
tags:
  - learning
  - dev/security
date: 2026-05-11
---
**Sources**: [IBM](https://www.ibm.com/think/topics/rbac)

**Related:** [[Security]], [[IAM]]

---

## Description

_Role-based access control (RBAC)_ is a model for **authorizing end-user access to systems**, applications and data **based on a user’s predefined role**.

For example, a security analyst can configure a firewall but can’t view customer data, while a sales rep can see customer accounts but can’t touch firewall settings.

By restricting users’ access to the resources needed for their roles, _RBAC_ can **help defend against malicious insiders**, **negligent employees** and **external threat actors**.

---

## Details

### Why is _RBAC_ important?

A role-based access control system enables organizations to take a granular approach to `IAM` while streamlining authorization processes and access control policies. Specifically, _RBAC_ helps organizations:

- Assign permissions more effectively
- Maintain compliance
- Protect sensitive data


### The three primary rules of _RBAC_

The National Institute of Standards and Technology (NIST), which developed the _RBAC_ model, provides three basic rules for all _RBAC_ systems.

1. **Role assignment:** A user must be assigned one or more active roles to exercise permissions or privileges.  

2. **Role authorization:** The user must be authorized to take on the role or roles they have been assigned.  

3. **Permission authorization:** Permissions or privileges are granted only to users who have been authorized through their role assignments.


### The four models of _RBAC_

There are four separate models for implementing _RBAC_, but each model begins with the same core structure. Each successive model builds new functionality and features upon the previous model.

- Core _RBAC_
- Hierarchical _RBAC_
- Constrained _RBAC_
- Symmetric _RBAC_

---

## Example

1. An IT administrator at a hospital creates an _RBAC_ role for “Nurse.”
2. The administrator sets permissions for the Nurse role, such as viewing medications or entering data into an electronic health record (EHR) system.
3. Members of the nursing staff at the hospital are assigned to the _RBAC_ Nurse role.
4. When users assigned to the Nurse role log-on, _RBAC_ checks which permissions they are entitled to and grants them access for that session.
5. Other system permissions such as prescribing medications or ordering tests are denied to these users because they are not authorized for the Nurse role.

---

## Claude Sessions
