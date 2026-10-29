# Linux File Permissions & Ownership Notes

## File / Directory Ownership

- **Ownership** identifies who created a file/directory and who can change its permissions.
- Creating a user automatically creates a matching **primary group** with the same name.
- The default primary group can be changed.

```bash
sudo groupadd admins
cat /etc/group | grep admins

# Change the primary group for a user
sudo usermod -g admins mostafa

cat /etc/passwd | grep mostafa   # primary group is now 'admins'

# Create another user to test
sudo useradd user1 -m -s /bin/bash   # -m: create home dir, -s: set shell
sudo passwd user1

su - user1

# Change ownership of a file (owner and/or primary group)
# chown newUser:newPrimaryGroup fileName   (group is optional)
sudo chown user1:user1 file1

# Change only the owner
sudo chown mostafa file1

# Change only the group
sudo chown :sami file1
# or
sudo chgrp admins file1
```

---

## The Three Main Permissions

| Permission | Symbol | Value | Meaning (file) | Meaning (directory) |
|------------|--------|-------|----------------|---------------------|
| Read | `r` | 4 | Read file content | List files/folders |
| Write | `w` | 2 | Write/modify content | Add files/folders |
| Execute | `x` | 1 | Run the file | `cd` into it |

Permissions are assigned to three categories:

- **User / Owner**
- **Group**
- **Others**

Each category has its own `rwx` permissions.

### Reading `ls -l` output

When listing with `ls -l`, you see **9 permission slots** (the *file mode*):

- First 3 → user
- Second 3 → group
- Third 3 → others

Each slot shows `r`, `w`, `x`, or `-` (no permission).

**Example:** `-rw-rw-r--`

- Leading `-` → it's a file (a `d` would mean directory)
- **User:** `rw-` → read & write, no execute
- **Group:** `rw-` → read & write, no execute
- **Others:** `r--` → read only

### Numeric values

- read + write = `4 + 2 = 6`
- full permissions = `rwx = 4 + 2 + 1 = 7`
- `rwxr-xr-x` = **755** (7: r+w+x, 5: r+x)
- `rw-r--r--` = **644**

---

## Changing Permissions with `chmod`

Symbols: user = `u`, group = `g`, others = `o`

```bash
sudo chmod u=rwx,g=rw,o=r fileName   # set permissions for each

# Remove read from others
sudo chmod u=rwx,g=rw,o-r fileName

# Add read for others
sudo chmod o+r file1

# Add execute for user and group at once
sudo chmod ug+x file1

# Numeric mode
chmod 644 file1
```

---

## Access Control List (ACL)

You can grant specific permissions to individual users/groups beyond the standard "others" category.

```bash
getfacl file1          # view the ACL

# output
# owner: user1
# group: admins
user::rw-
group::rw-
other::r--

getfacl file1 -t       # table format
```

Grant a specific user a specific permission:

```bash
setfacl -m u:user1:r file2   # give user1 read access on file2
```

After setting an ACL, `ls -l` shows a `+` at the end of the permission string:

```
rw- r-- ---+
```

The `+` indicates a special permission exists for a specific user.

---
