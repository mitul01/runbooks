# Managing MySQL Credentials Without Typing Passwords

This guide explains how to securely store and use MySQL credentials so you don’t have to enter passwords every time you connect with the mysql CLI.

MySQL provides a built-in tool called mysql_config_editor that stores credentials encrypted in ~/.mylogin.cnf

## 1. Verify Installation

Check if it’s available:
```bash
mysql_config_editor --help
```

## 2. Store Credentials

Set up a login path (you’ll be prompted for password once):
```bash
export DB_HOST=""
export DB_USER=""
mysql_config_editor set --login-path=${DB_HOST:?} \
  --host=${DB_HOST:?} \
  --user=${DB_USER:?} \
  --password
```

## 3. View Saved Paths
```bash
mysql_config_editor print --all
```

## 4. Connecting to database
```bash
mysql --login-path=${DB_HOST:?}
```
