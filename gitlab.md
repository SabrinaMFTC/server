## Summary

I. [Installing GitLab (Community Edition)](#i---installing-gitlab-community-edition)

II. [Configuring SMTP]()

---

# I - Installing GitLab (Community Edition)

1. **Access the desired VM** and follow the official quick installation guide from [GitLab's package repository](https://packages.gitlab.com/gitlab/gitlab-ce/install).

2. **After configuring the repository**, install GitLab Community Edition:

```bash
sudo apt update
sudo apt install gitlab-ce
```

3. **Check GitLab status** with:

```bash
sudo gitlab-ctl status
```

4. **Reconfigure GitLab** (apply changes and set up components):

```bash
sudo gitlab-ctl reconfigure
```

5. **Verify that GitLab is running** by accessing `http://<VM-IP>` via browser, or use the command below to test locally:

```bash
curl -I http://localhost
```

> ⚠️ GitLab uses its own Nginx on port 80. If another web server (like Nginx or Apache) is already running on that port, you must stop it or change GitLab’s port in `/etc/gitlab/gitlab.rb`.

6. **Set the root user password manually (if needed)** by entering the GitLab Rails console:

```bash
sudo gitlab-rails console
```

7. **Inside the console, run the following commands** to change the password:

```ruby
user = User.find_by(id: 1)
user.password = 'my-new-password'
user.password_confirmation = 'my-new-password'
user.save!
exit
```

8. **Log in to GitLab via browser** using the credentials:

```
Username: root
Password: <your-new-password>
```

---

# II. Configuring SMTP

## Using Gmail

1. Edit GitLab configuration file

```bash
sudo vim /etc/gitlab/gitlab.rb
```

2. Uncomment and edit the following lines

```rb
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.gmail.com"
gitlab_rails['smtp_port'] = 587
gitlab_rails['smtp_user_name'] = "<your-email@exemple.com>"
gitlab_rails['smtp_password'] = "<password>"
gitlab_rails['smtp_domain'] = "gmail.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false
gitlab_rails['smtp_pool'] = false

...

gitlab_rails['gitlab_email_from'] = '<your-email@exemple.com>>'
gitlab_rails['gitlab_email_display_name'] = 'GitLab HomeLab'
gitlab_rails['gitlab_email_reply_to'] = '<your-email@exemple.com>'
```

> When using Gmail service, you will need to generate a password for this purpose. You can access App passwords on your Gmail account Security and generate one for GitLab. You'll need to have 2-step verification on beforehand.

3. Now reconfigure GitLab with

```bash
sudo gitlab-ctl reconfigure
```

4. Access GitLab console

```bash
sudo gitlab-rails console
```

5. Enter

```rb
Notify.test_email('<your-email@exemple.com>', 'GitLab Test', 'This is a test email.').deliver_now
```

6. You should see a feedback message in the console and receive the email.

##

--- Cloudflare Tunnel ---

1. Install docker

https://docs.docker.com/engine/install/ubuntu/

2. c,loudflare > zero trust > networks > tunnels > selecionamos o tunnel já existente > configure > choose your env: docker > copy the command

```bash
docker run -d --name cloudflare --network host cloudflare/cloudflared:latest tunnel --no-autoupdate run --token <your-token>
```

execute no terminal

```bash
docker ps
```

na cloudflare > public hostnames > add a public hostname > subdomain (gitlab) > domain (smftc.cloud) > service: http > url: <ip-da-vm:port>
