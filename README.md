# Access Control Models

> Applied the Linux permission and ACL model hands-on, then designed a hybrid RBAC/ABAC access
> control policy for a hospital scenario.

*CNIT 271 — Cybersecurity Fundamentals II, Lab 1.*

## Summary

The practical half: users, groups, ownership, `chmod` permission bits, and fine-grained POSIX ACLs
on a Linux VM. The design half: a hybrid **Role-Based + Attribute-Based** access control model for a
hospital, with rules that depend on both a user's role and their attributes (department, clearance,
and relationship to the patient).

## Lab setup

- Windows 10 + Kali Linux VMs on Purdue's student cluster
- Linux CLI: `adduser`, `groupadd`, `usermod`, `chown`, `chmod`, `setfacl`, `getfacl`

## What I did &amp; found

**Permissions & ACLs**
- Created users and a group, set ownership with `chown`, and used `chmod 640` for
  owner-rw / group-r / other-none.
- Layered POSIX ACLs on top with `setfacl` to grant one user read/write and another read-only —
  more granular than the owner/group/other model — and verified with `getfacl`.
- Confirmed enforcement by logging in as each user and testing read vs write.

**Hybrid RBAC/ABAC hospital design**
- **Roles:** Doctor, Nurse, Administrator, Patient, Lab Staff
- **Objects:** Patient Records, Lab Results, Billing Records, Shift Schedule, Users Table
- **Attributes:** role, department, security clearance (user); object type, classification (object)
- **Relationships:** `PatientOf`, `NurseTo`, `Manages`
- **Sample rules:** doctors read/write only their own patients' records; lab staff access reports only
  when their clearance matches; doctors can't modify lab reports; admins manage user accounts.

Why hybrid: roles keep administration simple, attributes add context-sensitive granularity without
"role explosion."

## Screenshots

See [`screenshots/`](screenshots/) for user/group creation, the `chmod`/`setfacl`/`getfacl` output,
the read-vs-write enforcement test, and the hospital access-control diagram.

## Skills demonstrated

Linux permissions · POSIX ACLs · least privilege · RBAC · ABAC · access-control policy design
