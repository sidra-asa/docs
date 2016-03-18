> Author: sidra-asa

# Gitlab Migration

Here's the instruction to migrate gitlab from old one to new one.

## 1. Dump old data

    $ sudo service gitlab stop

    $ git clone https://github.com/gitlabhq/mysql-postgresql-converter.git -b gitlab
    $ cd mysql-postgresql-converter
    
    # The old database was MySQL.
    $ mysqldump --compatible=postgresql --default-character-set=utf8 -r gitlabhq_production.mysql -u root gitlabhq_production -p

    $ python db_converter.py gitlabhq_production.mysql gitlabhq_production.psql
    $ ed -s gitlabhq_production.psql < move_drop_indexes.ed
    
## 2. Install new Gitlab

    $ sudo apt-get install curl openssh-server ca-certificates postfix
    $ curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
    $ sudo apt-get install gitlab-ce

Change the external IP of gitlab server.

    $ vim /etc/gitlab/gitlab.rb
    
    # Modify the FQDN of gitlab.
    # Use https if you want to use ssl.
    external_url 'https://Gitlab_FQDN'
    
    # Install with Gitlab-CI
    ci_external_url 'https://Gitlab-CI_FQDN'
    gitlab_ci['gitlab_server'] = { "url" => 'https://Gitlab_FQDN', "app_id" => 'APP_ID', "app_secret" => 'APP_SECRET' }

    $ sudo gitlab-ctl reconfigure
    
The Gitlab should be ready.

## 3. Import the old data

    $ sudo gitlab-ctl stop
    $ sudo -u gitlab-psql  /opt/gitlab/embedded/bin/psql -f gitlabhq_production.psql -h /var/opt/gitlab/postgresql -d gitlabhq_production

Move the repos to New Gitlab.

    $ cp -r /GITLAB_HOME/repositories/* /var/opt/gitlab/git-data/repositories/*
    $ cp -r /GITLAB_HOME/gitlab-satellites/* /var/opt/gitlab/git-data/gitlab-satellites/*
    $ cp -r * /GITLAB_HOME/.ssh/authorized_keys /var/opt/gitlab/.ssh/authorized_keys
    $ chown -R /var/opt/gitlab/git-data/gitlab-satellites/*
    $ chown -R git:git /var/opt/gitlab/git-data/
    $ chown git:git /var/opt/gitlab/.ssh/authorized_keys
    
## 4. Change the owners of all tables and sqquences to gitlab

    $ for tbl in `psql -qAt -c "select tablename from pg_tables where schemaname = 'public';" YOUR_DB` ; do  psql -c "alter table \"$tbl\" owner to NEW_OWNER" YOUR_DB ; done
    $ for tbl in `psql -qAt -c "select sequence_name from information_schema.sequences where sequence_schema = 'public';" YOUR_DB` ; do  psql -c "alter table \"$tbl\" owner to NEW_OWNER" YOUR_DB ; done
    
    $ sudo -u git gitlab-rake db:migrate
    
Delete the table if there comes error like below:

     Mysql2::Error: Table 'users_groups' already exists: CREATE TABLE `users_groups` (`id` int(11) DEFAULT NULL auto_incremen
    
    # Drop the table;
    $ sudo -u gitlab-psql /opt/gitlab/embedded/bin/psql -h /var/opt/gitlab/postgresql -d gitlabhq_production
    =# drop table abuse_reports;
    
If there's no error at all, you can start gitlab now.

    $ sudo gitlab-ctl start

Enjoy it! 
