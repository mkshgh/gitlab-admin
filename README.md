# gitlab-admin

## Configuration
Already have ssl
```sh
external_url 'https://git.test.local'
letsencrypt['enable'] = false
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/git.test.local.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/git.test.local.key"
```

Use let's encrypt
```sh
# Domain should be pointed to the ip and port allowed
letsencrypt['enable'] = true
letsencrypt['contact_emails'] = ['user.name@email.com']
external_url 'https://git.test.local'
```


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


## All Repos
```sh
# needed deps
sudo apt install jq curl

# needed exports
export GITLAB_URL="https://git.test.local"
export TOKEN="glpat-XXXXXXXXXXXXXXXXXXXXXX"
```

```sh
#!/usr/bin/env bash
set -euo pipefail

page=1
count=0
: > repos.txt

while :; do
    r=$(curl -fsS --header "PRIVATE-TOKEN: $TOKEN" \
        "$GITLAB_URL/api/v4/projects?membership=false&owned=true&per_page=100&page=$page")

    n=$(jq 'length' <<< "$r")
    (( n == 0 )) && break

    jq -r '.[].path_with_namespace' <<< "$r" >> repos.txt
    count=$((count + n))
    ((page++))
done

echo "$count projects saved to repos.txt"
```
