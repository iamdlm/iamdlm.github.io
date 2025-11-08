---
title: "Setting Up PostgreSQL Backups on CentOS 7 with OneDrive"
date: 2025-01-08
draft: false
tags: ["postgres", "backup", "centos", "devops", "rclone"]
categories: ["infrastructure"]
---

Quick reference for setting up automated PostgreSQL backups on my CentOS 7 VPS with cloud sync to OneDrive.

## Overview

I have multiple apps running on a single PostgreSQL instance. Different apps need different backup frequencies, so I set up three schedules:
- **Daily**: 7-day retention
- **Weekly**: 4-week retention  
- **Monthly**: 3-month retention

Backups are stored locally on the VPS and synced to OneDrive using rclone.

## Initial Setup

### Install rclone

```bash
curl https://rclone.org/install.sh | sudo bash
```

### Configure OneDrive

Since the VPS is headless, I had to authorize rclone from my Windows machine:

**On Windows:**
```powershell
rclone authorize "onedrive"
```

This opens a browser for Microsoft authentication and outputs a token.

**On VPS:**
```bash
rclone config
```
- Choose new remote, name it `onedrive`
- Select Microsoft OneDrive storage type
- Leave client_id and client_secret blank (press Enter)
- Choose region: Microsoft Cloud Global
- When asked "Use auto config?", say **no**
- Paste the token from Windows
- When asked which drive, select **OneDrive (personal)**

Test the connection:
```bash
rclone lsd onedrive:
```

### Create Backup Directories

```bash
sudo mkdir -p /var/backups/postgres/{daily,weekly,monthly}
sudo chown {your_user}:{your_db_user} /var/backups/postgres -R
sudo chmod 750 /var/backups/postgres -R
```

## Backup Scripts

I created three scripts in `/usr/local/bin/` - one for each schedule.

### Daily Backup Script

`/usr/local/bin/backup-postgres-daily.sh`:

```bash
#!/bin/bash

BACKUP_DIR="/var/backups/postgres/daily"
RETENTION_DAYS=7
DATE=$(date +%Y%m%d_%H%M%S)
RCLONE_REMOTE="onedrive:Backups/Postgres"

# Change to a directory postgres user can access
cd /var/backups/postgres

# Daily databases
DATABASES=("app_1" "app_2" "app_3")

# Backup each database
for DB in "${DATABASES[@]}"; do
    echo "Backing up $DB..."
    sudo -u {your_db_user} pg_dump $DB | gzip > "$BACKUP_DIR/${DB}_${DATE}.sql.gz"
    
    if [ $? -eq 0 ]; then
        echo "✓ $DB backed up successfully"
    else
        echo "✗ $DB backup failed"
        exit 1
    fi
done

# Remove old backups (older than RETENTION_DAYS)
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete

# Sync to cloud
echo "Syncing daily backups to OneDrive..."
sudo -u {your_user} rclone sync "$BACKUP_DIR" "$RCLONE_REMOTE/daily" --log-file=/var/log/rclone-backup-daily.log

echo "Daily backup completed at $(date)"
```

### Weekly Backup Script

`/usr/local/bin/backup-postgres-weekly.sh`:

```bash
#!/bin/bash

BACKUP_DIR="/var/backups/postgres/weekly"
RETENTION_WEEKS=4
DATE=$(date +%Y%m%d_%H%M%S)
RCLONE_REMOTE="onedrive:Backups/Postgres"

# Change to a directory postgres user can access
cd /var/backups/postgres

# Weekly databases
DATABASES=("app_4" "app_5" "app_6")

# Backup each database
for DB in "${DATABASES[@]}"; do
    echo "Backing up $DB..."
    sudo -u {your_db_user} pg_dump $DB | gzip > "$BACKUP_DIR/${DB}_${DATE}.sql.gz"
    
    if [ $? -eq 0 ]; then
        echo "✓ $DB backed up successfully"
    else
        echo "✗ $DB backup failed"
        exit 1
    fi
done

# Remove old weekly backups (keep last 4 for each database)
for DB in "${DATABASES[@]}"; do
    ls -t "$BACKUP_DIR/${DB}_"*.sql.gz 2>/dev/null | tail -n +5 | xargs -r rm
done

# Sync to cloud
echo "Syncing weekly backups to OneDrive..."
sudo -u {your_user} rclone sync "$BACKUP_DIR" "$RCLONE_REMOTE/weekly" --log-file=/var/log/rclone-backup-weekly.log

echo "Weekly backup completed at $(date)"
```

### Monthly Backup Script

`/usr/local/bin/backup-postgres-monthly.sh`:

```bash
#!/bin/bash

BACKUP_DIR="/var/backups/postgres/monthly"
RETENTION_MONTHS=3
DATE=$(date +%Y%m%d_%H%M%S)
RCLONE_REMOTE="onedrive:Backups/Postgres"

# Change to a directory postgres user can access
cd /var/backups/postgres

# Monthly databases
DATABASES=("app_7" "app_8" "app_9")

# Backup each database
for DB in "${DATABASES[@]}"; do
    echo "Backing up $DB..."
    sudo -u {your_db_user} pg_dump $DB | gzip > "$BACKUP_DIR/${DB}_${DATE}.sql.gz"
    
    if [ $? -eq 0 ]; then
        echo "✓ $DB backed up successfully"
    else
        echo "✗ $DB backup failed"
        exit 1
    fi
done

# Remove old monthly backups (keep last 3 for each database)
for DB in "${DATABASES[@]}"; do
    ls -t "$BACKUP_DIR/${DB}_"*.sql.gz 2>/dev/null | tail -n +4 | xargs -r rm
done

# Sync to cloud
echo "Syncing monthly backups to OneDrive..."
sudo -u {your_user} rclone sync "$BACKUP_DIR" "$RCLONE_REMOTE/monthly" --log-file=/var/log/rclone-backup-monthly.log

echo "Monthly backup completed at $(date)"
```

### Make Scripts Executable

```bash
sudo chmod +x /usr/local/bin/backup-postgres-daily.sh
sudo chmod +x /usr/local/bin/backup-postgres-weekly.sh
sudo chmod +x /usr/local/bin/backup-postgres-monthly.sh
```

## Cron Schedule

Set up automated execution (use `nano` instead of vim):

```bash
export EDITOR=nano
sudo crontab -e
```

Add these lines:

```bash
# Daily backups at 2 AM
0 2 * * * /usr/local/bin/backup-postgres-daily.sh >> /var/log/postgres-backup-daily.log 2>&1

# Weekly backups on Sunday at 3 AM
0 3 * * 0 /usr/local/bin/backup-postgres-weekly.sh >> /var/log/postgres-backup-weekly.log 2>&1

# Monthly backups on 1st of month at 4 AM
0 4 1 * * /usr/local/bin/backup-postgres-monthly.sh >> /var/log/postgres-backup-monthly.log 2>&1
```

## Key Points to Remember

### Database Names
The `DATABASES` array must contain exact PostgreSQL database names. Check with:
```bash
psql -U {your_db_user} -l
```

### Empty Arrays
If a schedule has no databases, use an empty array: `DATABASES=()`  
Do NOT use `DATABASES=("")` - that will fail.

### User Context
- `pg_dump` runs as your database user: `sudo -u {your_db_user} pg_dump`
- `rclone` runs as your VPS user: `sudo -u {your_user} rclone`
- This is because rclone is configured for your user account, not root

### How It Works
1. Each database is backed up individually to local storage (not as one monolithic dump)
2. After ALL databases are backed up, rclone syncs to OneDrive
3. The sync is synchronous - the script waits for it to complete
4. Only new/changed files are uploaded (rclone is smart about this)

## Testing

Always test manually before trusting cron:

```bash
sudo /usr/local/bin/backup-postgres-daily.sh
sudo /usr/local/bin/backup-postgres-weekly.sh
sudo /usr/local/bin/backup-postgres-monthly.sh
```

Check logs:
```bash
cat /var/log/postgres-backup-daily.log
cat /var/log/rclone-backup-daily.log
```

Verify OneDrive:
```bash
rclone lsd onedrive:Backups/Postgres/daily
rclone lsd onedrive:Backups/Postgres/weekly
rclone lsd onedrive:Backups/Postgres/monthly
```

## Restoring a Backup

List available backups:
```bash
ls -lh /var/backups/postgres/daily/
```

Restore specific database:
```bash
gunzip -c /var/backups/postgres/daily/app_1_20250108_020001.sql.gz | psql -U {your_db_user} app_1
```

Or restore from OneDrive:
```bash
rclone copy onedrive:Backups/Postgres/daily/app_1_20250108_020001.sql.gz /tmp/
gunzip -c /tmp/app_1_20250108_020001.sql.gz | psql -U {your_db_user} app_1
```

## Final Structure

**Local:**
```
/var/backups/postgres/
├── daily/
│   └── app_1_YYYYMMDD_HHMMSS.sql.gz (7 days)
├── weekly/
│   └── app_2_YYYYMMDD_HHMMSS.sql.gz (4 weeks)
└── monthly/
    └── app_3_YYYYMMDD_HHMMSS.sql.gz (3 months)
```

**OneDrive:**
```
Backups/Postgres/
├── daily/
├── weekly/
└── monthly/
```

## Troubleshooting

If OneDrive sync fails, check:
1. Log file exists: `cat /var/log/rclone-backup-daily.log`
2. rclone works: `rclone lsd onedrive:`
3. Token hasn't expired: `rclone about onedrive:`
4. User context is correct (your VPS user has rclone configured)

That's it. Simple, reliable, automated backups with cloud redundancy.
