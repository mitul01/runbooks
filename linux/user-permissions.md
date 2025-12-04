# Linux User & Group Management Guide

This guide covers essential commands and best practices for managing users and groups in Linux systems.

## Table of Contents
- [User Management](#user-management)
- [Group Management](#group-management)
- [Permission Management](#permission-management)
- [Common Scenarios](#common-scenarios)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## User Management

### Creating Users

#### Create a basic user
```bash
# Create user with default settings
sudo useradd username

# Create user with home directory
sudo useradd -m username

# Create user with specific shell
sudo useradd -m -s /bin/bash username

# Create user with specific UID
sudo useradd -m -u 1050 username

# Create user with specific group
sudo useradd -m -g groupname username

# Create user with multiple groups
sudo useradd -m -G group1,group2,group3 username
```

#### Create user with full options
```bash
# Create user with complete configuration
sudo useradd -m -d /home/username -s /bin/bash -c "Full Name" -u 1050 -g primarygroup -G group1,group2 username
```

#### Set user password
```bash
# Set password interactively
sudo passwd username

# Set password non-interactively (scripting)
echo 'username:password' | sudo chpasswd

# Force password change on next login
sudo passwd -e username
```

### Viewing User Information

#### List users
```bash
# List all users
cat /etc/passwd

# List only usernames
cut -d: -f1 /etc/passwd

# List users with UID >= 1000 (regular users)
awk -F: '$3 >= 1000 {print $1}' /etc/passwd

# Show current user
whoami

# Show user ID information
id username
```

#### Detailed user information
```bash
# Show user account details
sudo chage -l username

# Show user's groups
groups username

# Show logged in users
who
w

# Show last login information
last username
```

### Modifying Users

#### Change user properties
```bash
# Change user's home directory
sudo usermod -d /new/home/path username

# Change user's shell
sudo usermod -s /bin/zsh username

# Change user's primary group
sudo usermod -g newgroup username

# Add user to additional groups
sudo usermod -aG group1,group2 username

# Replace user's secondary groups
sudo usermod -G group1,group2 username

# Change user's UID
sudo usermod -u 1100 username

# Lock user account
sudo usermod -L username

# Unlock user account
sudo usermod -U username

# Change user's comment/full name
sudo usermod -c "New Full Name" username
```

#### Password management
```bash
# Set password expiry
sudo chage -E 2024-12-31 username

# Set password change frequency (90 days)
sudo chage -M 90 username

# Set minimum password age (7 days)
sudo chage -m 7 username

# Set warning days before password expires
sudo chage -W 7 username

# View password aging information
sudo chage -l username
```

### Deleting Users

```bash
# Delete user (keep home directory)
sudo userdel username

# Delete user and home directory
sudo userdel -r username

# Force delete user (even if logged in)
sudo userdel -f username

# Delete user but backup home directory
sudo userdel --remove --backup-to /backup/users username
```

## Group Management

### Creating Groups

```bash
# Create a new group
sudo groupadd groupname

# Create group with specific GID
sudo groupadd -g 2000 groupname

# Create system group (GID < 1000)
sudo groupadd -r groupname
```

### Viewing Group Information

```bash
# List all groups
cat /etc/group

# List only group names
cut -d: -f1 /etc/group

# Show groups for current user
groups

# Show groups for specific user
groups username

# Show group members
getent group groupname
```

### Modifying Groups

```bash
# Add user to group
sudo usermod -aG groupname username
# OR
sudo gpasswd -a username groupname

# Remove user from group
sudo gpasswd -d username groupname

# Change group name
sudo groupmod -n newname oldname

# Change group GID
sudo groupmod -g 2100 groupname

# Set group administrator
sudo gpasswd -A username groupname
```

### Deleting Groups

```bash
# Delete group
sudo groupdel groupname

# Note: Cannot delete group if it's primary group for any user
```

## Permission Management

### File Ownership

```bash
# Change file owner
sudo chown username filename

# Change file owner and group
sudo chown username:groupname filename

# Change only group
sudo chgrp groupname filename

# Recursive ownership change
sudo chown -R username:groupname directory/

# Change ownership using numeric IDs
sudo chown 1000:1000 filename
```

### File Permissions

```bash
# Set permissions using symbolic notation
chmod u+rwx,g+rx,o+r filename

# Set permissions using octal notation
chmod 755 filename

# Common permission combinations
chmod 644 filename    # rw-r--r-- (files)
chmod 755 filename    # rwxr-xr-x (executables/directories)
chmod 600 filename    # rw------- (private files)
chmod 700 directory   # rwx------ (private directory)

# Recursive permission change
chmod -R 755 directory/

# Set special permissions
chmod u+s filename    # Set SUID
chmod g+s directory   # Set SGID
chmod +t directory    # Set sticky bit
```

### Access Control Lists (ACLs)

```bash
# View ACLs
getfacl filename

# Set ACL for user
setfacl -m u:username:rwx filename

# Set ACL for group
setfacl -m g:groupname:rx filename

# Set default ACL for directory
setfacl -d -m u:username:rwx directory/

# Remove ACL
setfacl -x u:username filename

# Remove all ACLs
setfacl -b filename

# Copy ACLs from one file to another
getfacl file1 | setfacl --set-file=- file2
```

## Common Scenarios

### Creating a Development User

```bash
#!/bin/bash
# Script to create a development user

USERNAME="devuser"
GROUPS="docker,sudo,developers"

# Create user with home directory
sudo useradd -m -s /bin/bash -c "Development User" $USERNAME

# Add to groups
sudo usermod -aG $GROUPS $USERNAME

# Set password
sudo passwd $USERNAME

# Create development directories
sudo mkdir -p /home/$USERNAME/{projects,scripts,logs}
sudo chown -R $USERNAME:$USERNAME /home/$USERNAME/

echo "Development user $USERNAME created successfully!"
```

### Setting Up a Shared Directory

```bash
#!/bin/bash
# Script to create a shared directory for a team

GROUP="team"
SHARED_DIR="/shared/team"

# Create group
sudo groupadd $GROUP

# Create shared directory
sudo mkdir -p $SHARED_DIR

# Set ownership and permissions
sudo chown root:$GROUP $SHARED_DIR
sudo chmod 2775 $SHARED_DIR  # SGID + rwxrwxr-x

# Set default ACL
sudo setfacl -d -m g::rwx $SHARED_DIR
sudo setfacl -d -m o::r-x $SHARED_DIR

echo "Shared directory $SHARED_DIR created for group $GROUP"
```

### Bulk User Creation

```bash
#!/bin/bash
# Script to create multiple users from a file

# Create users from list (format: username:fullname:group)
while IFS=: read -r username fullname group; do
    if [[ ! -z "$username" ]]; then
        echo "Creating user: $username"
        
        # Create user
        sudo useradd -m -s /bin/bash -c "$fullname" "$username"
        
        # Add to group if specified
        if [[ ! -z "$group" ]]; then
            sudo usermod -aG "$group" "$username"
        fi
        
        # Set temporary password
        echo "$username:TempPass123!" | sudo chpasswd
        
        # Force password change on first login
        sudo passwd -e "$username"
        
        echo "User $username created successfully"
    fi
done < users.txt
```

### User Audit Script

```bash
#!/bin/bash
# Script to audit user accounts

echo "=== USER AUDIT REPORT ==="
echo "Date: $(date)"
echo

echo "=== USERS WITH UID >= 1000 ==="
awk -F: '$3 >= 1000 {printf "%-15s UID: %-5s Shell: %-15s Home: %s\n", $1, $3, $7, $6}' /etc/passwd
echo

echo "=== USERS WITH EMPTY PASSWORDS ==="
sudo awk -F: '$2 == "" {print $1}' /etc/shadow
echo

echo "=== USERS WITH SUDO ACCESS ==="
getent group sudo | cut -d: -f4 | tr ',' '\n'
echo

echo "=== LOCKED ACCOUNTS ==="
sudo awk -F: '$2 ~ /^!/ {print $1}' /etc/shadow
echo

echo "=== PASSWORD EXPIRY INFORMATION ==="
for user in $(awk -F: '$3 >= 1000 {print $1}' /etc/passwd); do
    echo "User: $user"
    sudo chage -l "$user" | grep -E "(Last password change|Password expires)"
    echo
done
```

## Troubleshooting

### Common Issues

#### User cannot login
```bash
# Check account status
sudo passwd -S username

# Check if account is locked
sudo usermod -U username

# Check shell validity
grep username /etc/passwd
which /bin/bash

# Check home directory permissions
ls -ld /home/username
```

#### Permission denied errors
```bash
# Check file ownership
ls -la filename

# Check group membership
groups username

# Check ACLs
getfacl filename

# Verify sudo access
sudo -l -U username
```

#### Group membership not taking effect
```bash
# User needs to log out and back in, or use:
newgrp groupname

# Or restart the user's session
sudo su - username
```

### Recovery Procedures

#### Reset forgotten password
```bash
# Boot into single-user mode or use rescue disk
# Mount root filesystem
mount -o remount,rw /

# Reset password
passwd username

# Or remove password (temporarily)
passwd -d username
```

#### Fix corrupted user files
```bash
# Restore from /etc/skel
sudo cp /etc/skel/.* /home/username/
sudo chown -R username:username /home/username/
```

#### Recover from UID conflicts
```bash
# Find files owned by old UID
sudo find / -uid OLD_UID 2>/dev/null

# Change ownership to new UID
sudo find / -uid OLD_UID -exec chown NEW_UID:NEW_GID {} \; 2>/dev/null
```

### Useful Log Locations

```bash
# Authentication logs
/var/log/auth.log          # Debian/Ubuntu
/var/log/secure           # RedHat/CentOS

# User activity logs
/var/log/utmp             # Currently logged in users
/var/log/wtmp             # Login/logout history
/var/log/btmp             # Failed login attempts

# Sudo logs
/var/log/sudo.log         # Sudo command history
```

### Emergency Access

```bash
# Create emergency admin account
sudo useradd -m -s /bin/bash -G sudo emergency_admin
sudo passwd emergency_admin

# Grant temporary sudo access
echo "username ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/temp_access

# Remove when no longer needed
sudo rm /etc/sudoers.d/temp_access
```