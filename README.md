
# THE PROJECT IS ARCHIVED

This plugin is maintained officially by The Foreman and will not got update.

Official Foreman Plugin: https://github.com/theforeman/foreman_expire_hosts

---

# foreman\_expire\_hosts

# Context

Foreman makes host creation extremely simple for a sysadmin or a user.  However this simplicity lead to wasteful usage of compute resources. foreman_expiry plugin allows to specify an expiry date for the host. On this date the host will be deleted.

This plugin add expired on(date) field to host form under Additional Information section. If we create any host with expiry date, then that host will be stopped on given date and then deleted. If host has null/blank for expired on field then that host will be live forever (until it deleted manually).

This plugin will send two warning notification before host expiry (see settings.yaml). It also sends notifications when the host stopped on its expiry date and when host is deleted after few days (configured in settings).

# Screenshots
![Expiry date field in host form](https://raw.githubusercontent.com/ingenico-group/screenshots/master/foreman_host_expiry/expiry-date-field-in-host-form.png)

![Expiry date field in host show page](https://raw.githubusercontent.com/ingenico-group/screenshots/master/foreman_host_expiry/expiry-date-in-host-show-page.png)

![Plugin Settings](https://raw.githubusercontent.com/ingenico-group/screenshots/master/foreman_host_expiry/settings.png)


# Installation
Please see the Foreman manual for appropriate instructions:

* [Foreman: How to Install a Plugin](http://theforeman.org/manuals/latest/index.html#6.Plugins)

Example installation from source.
Require foreman_expire_hosts gem installation (edit `~foreman/bundler.d/Gemfile.local.rb`):

```yaml
gem 'foreman_expire_hosts', :git => "https://github.com/ingenico-group/foreman_expire_hosts.git"
```

Update Foreman with the new gems:

    bundle update foreman_expire_hosts

# Dependency

This gem required uglifier, execjs and therubyracer gems to load assets. If you install this plugin through RPM package, please install ruby193-rubygem-uglifier, ruby193-rubygem-execjs and ruby193-rubygem-therubyracer rpm package before installing this plugin RPM package

# Post installation

This plugin needs additional column in hosts table. Please run migration with below command

```yaml
VERSION=20150427101516 RAILS_ENV=production rake db:migrate:up
```

Add below line to crontab under root user to take appropriate action on expiring/expired hosts and notify user about those hosts. This cronjob will run at 11:30 PM(Midnight)


```
30 23 * * *  /usr/sbin/foreman-rake expired_hosts:deliver_notifications >> /var/log/foreman/expired_hosts.log 2>&1
```

# Pre remove

If we want to remove this feature and remove expired_on column from hosts table, please run below rake task and remove this gem from the Gemfile

```yaml
VERSION=20150427101516 RAILS_ENV=production rake db:migrate:down
```

# Configuration

This plugin will add configurations to settings table and are editable from settings page

![Plugin Settings](https://raw.githubusercontent.com/ingenico-group/screenshots/master/foreman_host_expiry/settings.png)

NOTE: After installing this plugin, please update administrator email in Foreman Web UI (More -> Settings -> General) with valid email. This can be used to send notification when plugin failed to deliver notifications to its owner.


# Foreman API to add expiry date to host

Existing foreman host create/edit API can be used to add/update host's expiry date. For example

Create host with expiry date

```yaml
POST /api/hosts
{
  "host": {
    "name": "testhost11",
    "environment_id": "334344675",
    "domain_id": "22495316",
    "ip": "10.0.0.20",
    "mac": "52:53:00:1e:85:93",
    "ptable_id": "980190962",
    "medium_id": "980190962",
    "architecture_id": "501905019",
    "operatingsystem_id": "1073012828",
    "puppet_proxy_id": "980190962",
    "compute_resource_id": "980190962",
    "root_pass": "xybxa6JUkz63w",
    "location_id": "255093256",
    "organization_id": "447626438",
    "expired_on": "30/12/2014"  # dd/mm/yyyy format
  }
}
```

curl command example

```yaml
curl -u admin:changeme 'https://your-foreman-url.com/api/hosts' -d 'host[name]=testhost11&host[expired_on]=30/12/2014&......' -X POST
```
Update host expiry date

```yaml
POST /api/hosts/testhost11
{
  "host": {
    "expired_on": "30/12/2014"  # dd/mm/yyyy format
  }
}
```

curl command example

```yaml
curl -u admin:changeme 'https://your-foreman-url.com/api/hosts/testhost11' -d 'host[expired_on]=30/12/2014' -X PUT
```

## License

GPLv3
