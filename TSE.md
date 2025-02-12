
### **1. Usernames and Home Directories Script**
 **Script to list users & their home directories**
 *list_users.sh script contents*
 ```
#!/bin/bash
# List all users in the system and their home directories
  cat /etc/passwd | awk -F: '{print $1 ":" $6}' 
```

 **Monitor user changes**
 *monitorusers.sh script contents*
```
#!/bin/bash
# Generate MD5 hash of users
USER_HASH=$(awk -F: '{print $1 ":" $6}' /etc/passwd | md5sum | awk '{print $1}')
LOG_FILE="/var/log/current_users"
CHANGE_LOG="/var/log/user_changes"

# Confirm log file existence
if [[ ! -f "$LOG_FILE" ]]; then
    echo "$USER_HASH" > "$LOG_FILE"
    exit 0
fi

# Read old hash
OLD_HASH=$(cat "$LOG_FILE")

# Compare hashes
if [[ "$USER_HASH" != "$OLD_HASH" ]]; then
    echo "$(date '+%Y-%m-%d %H:%M:%S') changes occurred" >> "$CHANGE_LOG"
    echo "$USER_HASH" > "$LOG_FILE"
fi
```

#### **Cron entry to run scripts hourly**
```
0 * * * * /home/isaac/list_users.sh | /home/isaac/monitorusers.sh
```

---

## Possible causes of slowness in the web App
Given that the application runs on a single Linux machine with 8GB RAM, 2 CPU cores, and SSD storage, the slowness could stem from several factors:

1. Application-Level Issues
Inefficient queries: If the application is making SQL queries that are not optimized (e.g. SELECT * instead of selecting required columns), it can slow down page loads.
Heavy computation on requests: There may be high response times if the application performs extensive computations in the request-response cycles.
2. Database performance bottlenecks
Connection pooling Issues: In case the database is configured with too few connections and database requests queue up, this could lead to slowness.
3. Web server traffic and API response times
Limited CPU resources: With only 2 CPU cores in the given server, it may struggle under an increased concurrent traffic.
Slow external API calls: In case the web page depends on external API calls, for say fetching third-party data etc, slow responses from those APIs can cause delays in processing pages.
4. System-level issues
RAM Usage: If the system runs out of available memory, it may start swapping to disk, significantly reducing performance.
CPU Overload: A prolonged high CPU utilization can slow down request processing, increase response times, and cause overall system lag.

### **Troubleshooting steps I will take:**
- **Check Server Resource Utilization:**
   ```
   top      # Monitor CPU usage. or >htop
   free -h        # Check RAM usage
   df -hT              # Check disk utilization
   ps -ef --sort=-%cpu | head        # Monitor processes' resource utilization 
   ```
- **Analyze any slow db queries:**
   ```
   mysql -u root -p -e 'SHOW PROCESSLIST;'
   ```
   
- **Check database status:**
   ```
    systemctl status mysql  # or systemctl status postgresql
   ```  
- **Check connectivity to database:**
   ```
   telnet <database_host> <db-port>  # eg. telnet 192.168.1.100 3306
   ```  
  
   
- **Check web server Logs:**
   ```
   tail -f /var/log/nginx/*.log
   tail -f /var/log/httpd/*.log
   ```
- **Optimize system performance:**
   - `tuned-adm active` (view the active profile)
   - `tuned-adm recommend` (view the recommended profile)
   - `tuned-adm active hpc-compute` (set a recommended profile)

- **Check on SELinux alarms:**
   - `sealert -a /var/log/audit.log` 

---

## **3. Git commit graph**
*Steps*
```
#Create a working directory and cd into it
mkdir gitwork
cd gitwork

#Initialize a new git repository
git init

# Create the first commit
echo "First commit" > file.txt
git add file.txt
git commit -m "first commit"

# Create the second commit
echo "Second commit" >> file.txt
git commit -am "second commit"


# Create a new branch and add a commit
git checkout -b feature-branch
echo "Awesome feature" > feature.txt
git add feature.txt
git commit -m "awesome feature"

# 6. Switch back to main and add another commit
git checkout master
echo "Third commit" >> file.txt
git commit -am "third commit"

# 7. Merge feature-branch into main (creating a merge commit)
git merge feature-branch  -m "Merge"

# Create the fourth commit
echo "fourth commit" >> file.txt
git commit -am "fourth commit"

#Display a graphical commit history
git log --all --decorate --oneline --graph
```

---

# Using Git to Implement a New Feature Without Affecting the Main Branch

Git is a version control tool for software development, that helps teams to collaborate efficiently while tracking changes in their code. One of its best practices is working on new features or changes while not affecting the master branch. This guide will help you on how to achieve this using feature branches in Git.

 **Why Use a feature branch?**
Working directly on the `master` branch while developing a new feature is risky because:
- Your current/work-in-progress code might fail the application.
- Other teams rely on `master` branch stability.
- It may take a long time to track individual changes and rollback if needed.

By using feature branches, you isolate your work and makes it easier to test and review, and merge only when ready.

## Step-by-Step Guide

**1. Clone the repo**
```
git clone <repository-url>
cd <repo-name>
```
This command pulls/downoads the latest version of the repository into your local machine.

**2. Create a new feature branch**
Create a new branch for your feature before making changes.
```
git checkout -b feature-new-update
```
- `git checkout -b` creates and switches to the new branch `feature-new-update`.

**3. Make your code changes**
Make your changes now that you are on the feature branch.

```
echo "New feature implementation" > feature.txt
```
Add and commit your changes:
```
git add feature.txt
git commit -m "Added feature implementation"
```

**4. Push the feature branch to GitLab**
Push the branch to save your changes in GitLab
```
git push origin feature-new-update
```
This will make your feature branch available remotely for collaboration.

**5. Creating a merge request on GitLab**
In GitLab, navigate to your repository and create a Merge Request (MR):
- Select `feature-new-update` as the source branch.
- Select `master` as the target branch.
- Add a description of the feature and request a review.

**6. Review and merge the feature**
Upon feature approval:
```
git checkout main
git pull origin main  # Ensure you have the latest version
git merge feature-new-update
```
This will merge your feature into the main branch.

**7. Delete the feature ranch optionally**
You can delete the feature branch, after merging, to keep things clean:
```
git branch -d feature-new-update
```
Delete it from GitLab as well:
```
git push origin --delete feature-new-update
```

By using feature branches, developers can safely work on changes without disrupting the main application.

---


## **5. Technical Resource Review**
A tool that I recently came across and enjoyed learning is InstructLab, an open-source initiative developed by IBM and Red Hat to enhance Large Language Models (LLMs) and AI adoption.

Review:
I found InstructLab particularly interesting because it is beginner-friendly and removes barriers like high costs and complex hardware requirements, making AI experimentation accessible for everyone.

What I loved:
InstructLab can be installed in linux as a package, and developers can easily import available models, fine-tune them and develop applications easily.

What could be improved:
Limited adoption – Since InstructLab is a relatively new technology, it lacks a sufficient ecosystem, making it difficult to find community support or case studies.

References:
Website: https://instructlab.ai
Documentation: https://docs.instructlab.ai

---


