## How to Manually Queue a Notification in Staging or Production
Say you made a new notification that you call somewhere with `queue_notification`. This is a tutorial on how to queue such a notification in staging/production to see if it actually works end to end. 

1. ssh into bastion ex: `ssh elena@bastion1.tilt.com`
 - Need access to bastion? Make a PR like this in the chef repo to add your ssh key: https://github.com/tilteng/chef/pull/1886
 - How to get your public ssh key? https://help.github.com/articles/checking-for-existing-ssh-keys/
2. `api production list` (`api staging list` for staging)
 - This might take ~10 second to fully load
 - Find the api box close to your region (will look like this in the list: `staging-api-08f2eaf873472058d: 10.20.30.91` or `production-api-0ff951a1820c7a1b1: 10.20.51.24`) 
3. ssh into that api box ex: `ssh elena@10.20.51.24`
4. Now you'll set up some environments, make a script, and run it  
```
sudo su - 

export DANCER_ENVIRONMENT=production
export PERL5LIB=/srv/api/current/lib:/srv/api/current/locallib/lib/perl5

cd /srv/api/current/bin

touch manual_notif.pl
vim manual_notif.pl
```
This is your script where you'll queue up your notification. Here's an example script for inspiration/template:
```
package bin::Notifs;

use v5.14;
use FindBin qw($RealBin);
use lib "$RealBin/../lib";

use Dancer qw(:script);
use Crowdtilt::Internal::API;
use Crowdtilt::Internal::Common qw(queue_notification smart_rset);

queue_notification {
    action                      => 'on_complete_bank_setup',
    user_id                     => 'USR9EDDCBEA37B411E4B88D10017B95D8DD',
    amount                      => 1000,
    kyc_needed		              => 1,
    use_handlers                => 1,
};

1;
```
Alright, that's it! now run it!~ 
`cd /srv/api/current`
`perl bin/manual_notif.pl`  

Check mc.tilt.com or mc.staging.tilt.com to see your notification. For staging since no notifications _actually_ get sent, you'll have to check the logs, and really the only useful one to check is Email as you might see `<missing_vars><![CDATA[email,template_id]]></missing_vars>` error on your template. 

Metrics/Analytics/Logging/Error tracking for your new notification:
- [Rollbar](https://rollbar.com/tilt/api/items/) to see if your notification caused any errors 
- [DataDog](https://app.datadoghq.com/dash/171894/notifications?live=true&page=0&is_auto=false&from_ts=1471797252657&to_ts=1471883652657&tile_size=m#) to see general analytics on notifications


 
