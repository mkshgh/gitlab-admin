# gitlab-admin

## Simple Tasks
```sql
# Go to DB
sudo gitlab-psql -d gitlabhq_production

# Check users
\d users
SELECT id, username, email, state, admin FROM users ORDER BY id;

# Block all except root
UPDATE users
SET state = 'blocked'
WHERE username <> 'root';
\d application_settings
```

```ruby
#existing user
sudo gitlab-rails console
user = User.find_by(username: 'root')
user.password = 'YourNewStrongPassword!'
user.password_confirmation = 'YourNewStrongPassword!'
user.save!

#New User
sudo gitlab-rails runner "user = User.new(username: 'root', name: 'Root Admin', email: 'admin@test.local', password: 'YourNewStrongPassword!'); user.admin = true; user.skip_confirmation!; user.save!"

```

## Backup/Restore
```sh
#/bin/sh
#cd /var/opt/gitlab/backups/
BACKUP_ID="2792438546_2026_10_24_16.4.3"
gitlab-ctl stop puma
gitlab-ctl stop sidekiq

echo ""
echo "[1/6] Restoring GitLab backup..."
gitlab-rake gitlab:backup:restore BACKUP="$BACKUP_ID"
#gitlab-backup restore BACKUP="$BACKUP_ID"
echo ""
echo "[2/6] Reconfiguring GitLab..."

gitlab-ctl reconfigure

echo ""
echo "[3/6] Starting GitLab..."

gitlab-ctl start

echo ""
echo "[4/6] Checking GitLab status..."

gitlab-ctl status

echo ""
echo "[5/6] Running GitLab check..."

gitlab-rake gitlab:check SANITIZE=true

echo ""
echo "[6/6] Restore completed successfully."
```
