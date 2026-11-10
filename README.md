# Access Control Models

> Built a Windows 10 and Kali Linux lab on Nutanix, applied the Linux permission and ACL model
> hands-on, then designed a hybrid RBAC/ABAC access-control policy for a hospital.

*CNIT 271 (Cybersecurity Fundamentals II), Lab 1.*

## Lab setup

Both VMs were built from scratch on Purdue's Nutanix student cluster:

| VM | vCPU / RAM / Disk | Network | Boot & security |
| --- | --- | --- | --- |
| Windows 10 (`CNIT271_G37.jgurtW`) | 1 CPU × 2 cores / 2 GiB / 48 GiB SATA | 44.67.37.22 | UEFI, **Secure Boot + vTPM** |
| Kali Linux (`CNIT271_G37.jgurtKL`) | 1 CPU × 2 cores / 2 GiB / 48 GiB SATA | 44.67.37.20 | UEFI, **vTPM** |

## Procedure

### 1. Windows 10 VM
1. Created the VM with memory overcommit on, and attached two CD-ROMs cloned from images: the Windows 10
   ISO and **Nutanix VirtIO**.
2. Booted with **F12 → Boot Manager → UEFI QEMU DVD-ROM**. Chose **Custom install** and loaded the
   **VirtIO storage driver** (Windows 10 x64) so the installer could see the virtual disk.
3. Finished setup, created a local user, then ran `Nutanix-VirtIO-1.2.5-x64` from the mounted drive to
   install the rest of the paravirtual drivers.

### 2. Kali Linux VM
1. Same VM build with the Kali image attached, timezone UTC.
2. Graphical install: set hostname, root and user accounts, **guided partitioning** on the whole disk,
   and the Xfce desktop.

### 3. Linux permissions and ACLs
```bash
# Users and group
sudo adduser user1; sudo adduser user2; sudo adduser user3
sudo groupadd labgroup
sudo usermod -aG labgroup user1
sudo usermod -aG labgroup user2

# File ownership and base permissions: owner rw, group r, other none
touch secret.txt
sudo chown user1:labgroup secret.txt
sudo chmod 640 secret.txt
ls -l secret.txt

# POSIX ACLs for per-user exceptions on top of owner/group/other
sudo setfacl -m u:user2:rw secret.txt
sudo setfacl -m u:user3:r  secret.txt
getfacl secret.txt
```

Then tested enforcement by signing in as the read-only user. `cat secret.txt` worked, and saving
changes with `nano` was denied. *(`chmod o+x` was also needed before the read-only users could get
to the file. On a directory, the execute bit is what allows a user to enter it.)* User and group names
above are placeholders.

### 4. Hybrid RBAC/ABAC design for a hospital

| Element | Defined as |
| --- | --- |
| **Subjects (roles)** | Doctor, Nurse, Administrator, Patient, Lab Staff |
| **Objects** | Patient Records, Lab Test Results, Billing Records, Shift Schedule, Users Table |
| **Rights** | Read, Write, Modify, Delete, Execute |
| **User attributes** | Role, Department (Surgery / Administration / Laboratory), Security Clearance |
| **Object attributes** | ObjectType (PatientRecord / LabReport / Bill), Classification (Public / Confidential), EmployeeID, PatientID |
| **Relationships** | `PatientOf(PatientID, Doctor/Nurse)`, `NurseTo(Nurse, Patient/Doctor)`, `Manages(Admin, UsersTable)` |

**Access rules**
1. Doctors can read/write **only their own patients'** records (`PatientOf`).
2. Nurses can modify records **only for patients they're assigned to** (`NurseTo`).
3. Lab Staff can read/write a lab report only while working on it, and only when their **clearance
   matches** the report's classification.
4. Doctors **cannot modify** lab reports. Clinical staff can't change lab results.
5. Administrators can add, modify and delete user accounts (`Manages`) but get no implied access to
   clinical data.

**Why hybrid:** Roles alone would need a separate role for every doctor-patient pair ("role explosion").
Attributes alone are hard to administer. Using roles for broad access and attributes and relationships
for context keeps it both manageable and least-privilege.

## Troubleshooting
The Windows boot menu listed several devices, and it wasn't obvious which one was the installer. I tried
them in order. A wrong choice dropped back to the menu, and **UEFI QEMU DVD-ROM QM00003** was the installer.

## Skills demonstrated

Nutanix VM provisioning · VirtIO drivers · Secure Boot / vTPM · Linux users & groups · `chmod` permission
bits · POSIX ACLs (`setfacl` / `getfacl`) · least privilege · RBAC · ABAC · access-control policy design

## Screenshots

See [`screenshots/`](screenshots/) for the VM configs, `chmod`/`setfacl`/`getfacl` output, the read-vs-write
test, and the hospital access-control diagram.
