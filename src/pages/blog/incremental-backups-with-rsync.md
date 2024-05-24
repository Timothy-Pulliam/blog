---
layout: "../../layouts/BlogPost.astro"
title: "Incremental Backups with rsync"
description: "Incremental Backups with rsync"
pubDate: "May 24 2024"
heroImage: "/linux-hero.jpg"
previewText: "A backup and restore script for incremental rsync backups"
---

Below is an example of a script that will perform regular full backups (e.g., weekly) and daily incremental backups using `rsync`. This script uses hard links to make incremental backups efficient in terms of storage space.

### Script Explanation

- **Full Backup**: Performed once a week.
- **Incremental Backups**: Performed daily, linking to the latest backup (either full or incremental).

### Script

```bash
#!/bin/bash

# Configuration
BACKUP_SRC="/path/to/source"  # e.g. /home/
BACKUP_DEST="/path/to/backup" # e.g. /net/nfs-server.example.com/backups/home/
FULL_BACKUP_DIR="$BACKUP_DEST/full"
INCREMENTAL_BACKUP_DIR="$BACKUP_DEST/incremental"
DATE=$(date +%Y-%m-%d)
DAY_OF_WEEK=$(date +%u)  # 1 (Monday) to 7 (Sunday)
FULL_BACKUP_DAY=7        # Perform full backup on Sunday

# Create directories if they don't exist
mkdir -p "$FULL_BACKUP_DIR"
mkdir -p "$INCREMENTAL_BACKUP_DIR"

# Determine the type of backup to perform
if [ "$DAY_OF_WEEK" -eq "$FULL_BACKUP_DAY" ]; then
  BACKUP_TYPE="full"
  NEW_BACKUP_DIR="$FULL_BACKUP_DIR/$DATE"
else
  BACKUP_TYPE="incremental"
  NEW_BACKUP_DIR="$INCREMENTAL_BACKUP_DIR/$DATE"
  LATEST_BACKUP=$(ls -td $FULL_BACKUP_DIR/* $INCREMENTAL_BACKUP_DIR/* 2>/dev/null | head -1)
fi

# Perform the backup
echo "Starting $BACKUP_TYPE backup for $DATE..."

if [ "$BACKUP_TYPE" = "full" ]; then
  rsync -a --delete "$BACKUP_SRC/" "$NEW_BACKUP_DIR/"
else
  rsync -a --delete --link-dest="$LATEST_BACKUP" "$BACKUP_SRC/" "$NEW_BACKUP_DIR/"
fi

echo "$BACKUP_TYPE backup completed for $DATE."

# Optional: Clean up old backups (e.g., older than 30 days)
find "$FULL_BACKUP_DIR"/* -maxdepth 0 -type d -mtime +30 -exec rm -rf {} \\;
find "$INCREMENTAL_BACKUP_DIR"/* -maxdepth 0 -type d -mtime +30 -exec rm -rf {} \\;

echo "Old backups cleaned up."
```

### How to Use the Script

1. **Configuration**:
   - Set `BACKUP_SRC` to the path you want to back up.
   - Set `BACKUP_DEST` to the directory where you want to store the backups.
2. **Schedule the Script**:
   - Use `cron` to schedule the script to run daily. Edit the crontab with `crontab -e` and add an entry:
     ```bash
     0 2 * * * /path/to/backup-script.sh
     ```

### Notes

- **Full Backup Day**: The script is set to perform a full backup on Sunday (`DAY_OF_WEEK` 7). You can change this by modifying the `FULL_BACKUP_DAY` variable.
- **Hard Links**: Incremental backups use hard links to the previous backup to save space, storing only the differences.
- **Clean Up**: The script includes a clean-up step to remove backups older than 30 days. Adjust this retention period as needed.

By using this script, you can automate the process of creating regular full backups and daily incremental backups efficiently with `rsync`.

# Restoring from Incremental Backups

Below is a script that restores data from a set of full and incremental `rsync` backups. This script will find the latest full backup and then apply all subsequent incremental backups in order to reconstruct the desired state.

### Assumptions:

- Full backups are stored in `/backup/full/YYYY-MM-DD`.
- Incremental backups are stored in `/backup/incremental/YYYY-MM-DD`.
- The restore process applies the full backup first, followed by incremental backups in chronological order.

### Restore Script

```bash
#!/bin/bash

# Configuration
BACKUP_DEST="/restore/destination"
FULL_BACKUP_DIR="/backup/full"
INCREMENTAL_BACKUP_DIR="/backup/incremental"

# Find the latest full backup
LATEST_FULL_BACKUP=$(ls -td "$FULL_BACKUP_DIR"/* | head -1)

if [ -z "$LATEST_FULL_BACKUP" ]; then
  echo "No full backups found."
  exit 1
fi

# Find all incremental backups since the latest full backup
INCREMENTAL_BACKUPS=$(find "$INCREMENTAL_BACKUP_DIR" -type d -newer "$LATEST_FULL_BACKUP" | sort)

echo "Starting restoration process..."
echo "Restoring from full backup: $LATEST_FULL_BACKUP"

# Step 1: Restore the full backup
rsync -a --delete "$LATEST_FULL_BACKUP/" "$BACKUP_DEST/"

# Step 2: Apply incremental backups in order
for INCREMENTAL_BACKUP in $INCREMENTAL_BACKUPS; do
  echo "Applying incremental backup: $INCREMENTAL_BACKUP"
  rsync -a --delete "$INCREMENTAL_BACKUP/" "$BACKUP_DEST/"
done

echo "Restoration complete."
```

### How to Use the Script

1. **Save the Script**: Save the script to a file, for example, `/usr/local/bin/restore-backup.sh`.
2. **Make the Script Executable**:

   ```bash
   sudo chmod +x /usr/local/bin/restore-backup.sh
   ```

3. **Run the Script**:

   ```bash
   sudo /usr/local/bin/restore-backup.sh
   ```

### Detailed Explanation:

1. **Configuration**:
   - `BACKUP_DEST`: The directory where the data will be restored.
   - `FULL_BACKUP_DIR`: Directory containing full backups.
   - `INCREMENTAL_BACKUP_DIR`: Directory containing incremental backups.
2. **Finding the Latest Full Backup**:
   - The script uses `ls -td "$FULL_BACKUP_DIR"/* | head -1` to find the most recent full backup directory.
3. **Finding Incremental Backups**:
   - The script uses `find "$INCREMENTAL_BACKUP_DIR" -type d -newer "$LATEST_FULL_BACKUP" | sort` to find and sort all incremental backups created after the latest full backup.
4. **Restoring Full Backup**:
   - The script uses `rsync -a --delete "$LATEST_FULL_BACKUP/" "$BACKUP_DEST/"` to restore the full backup.
5. **Applying Incremental Backups**:
   - The script iterates over each incremental backup and applies it using `rsync -a --delete "$INCREMENTAL_BACKUP/" "$BACKUP_DEST/"`.

### Notes:

- **Ensure Consistency**: Before running the restore script, ensure no backup processes are running to avoid data inconsistency.
- **Permissions**: Make sure the user running the script has the necessary permissions to read from the backup directories and write to the restore destination.
- **Backup Verification**: After the restore process, verify the restored data to ensure its integrity and completeness.

By using this script, you can automate the restoration process from a series of full and incremental backups created by `rsync`.
