# ![Cuttlefish](https://raw.github.com/mlandauer/cuttlefish/master/app/assets/images/cuttlefish_80x48.png) Cuttlefish for officee haq haq haq

[![Build Status](https://travis-ci.org/mlandauer/cuttlefish.svg?branch=master)](https://travis-ci.org/mlandauer/cuttlefish) [![Coverage Status](https://coveralls.io/repos/github/mlandauer/cuttlefish/badge.svg?branch=master)](https://coveralls.io/github/mlandauer/cuttlefish?branch=master) [![Maintainability](https://api.codeclimate.com/v1/badges/abe94fb0811e8e8c512a/maintainability)](https://codeclimate.com/github/mlandauer/cuttlefish/maintainability)

* Project site: [cuttlefish.io](https://cuttlefish.io)
* Github repo:  [github.com/mlandauer/cuttlefish](https://github.com/mlandauer/cuttlefish)

Cuttlefish is a lovely, easy to set up transactional email server

Sending a few emails from your app is easy. Sending lots becomes painful. There are so many hidden gotchas. Do your emails get delivered? Are you being considered a spammer? What about all those bounced emails?

Let's make sending lots of emails fun again!

And without the hidden dangers of vendor lock in of commercial transactional email services.

* Send email from your application using smtp in the usual way and get all sorts of added benefits for no effort
* A lovely web UI to browse what's happening
* Monitor in real time which emails arrive at their destination and which bounce
* Works with any web framework and language
* Automatically not send emails to destinations that have hard bounced in the past
* Track which emails are opened and which links are clicked
* Statistics on emails sent, soft/hard bounced and held back
* View the full email content for recently sent emails
* Multiple applications can each have their own SMTP authentication
* [GraphQL](https://graphql.org/) API where anything you can do in the admin UI can do with the API
* Web callbacks on successful or failed deliveries of emails
* Check your IP reputation with one click
* Easy to install and get going quickly
* Built in, super easy to set up, automatic DKIM signing
* Postfix, which you know and trust, handles email delivery
* Open source, so no vendor lock in.

Cuttlefish is in beta. It's been used in production by [OpenAustralia Foundation](http://www.openaustraliafoundation.org.au)'s projects for several years and has sent many millions of emails.

## Screenshots

![Sign up](https://raw.github.com/mlandauer/cuttlefish/master/app/assets/images/screenshots/1.png)
![Dashboard](https://raw.github.com/mlandauer/cuttlefish/master/app/assets/images/screenshots/2.png)
![Email](https://raw.github.com/mlandauer/cuttlefish/master/app/assets/images/screenshots/3.png)

## Things on the cards

* REST API for deep integration with your application
* Web callbacks on successful delivery, hard bounces, open and click events
* "out of office" and bounce reply filtering
* Incoming email

## Dependencies
Ruby 2.5.1, PostgresQL, Redis (2.4 or greater), Postfix

Also you need the following libraries:
imagemagick, libmagickwand-dev, libpq-dev

For development, however, the only dependencies are Docker and Docker compose.

## Development

Setting up a local development environment with all the correct dependencies and
moving parts is now very straightforward by using [Docker](https://www.docker.com/).

To start with:
```
docker-compose run web bundle exec rake db:create db:schema:load
```

Now add some example seed data. This will also create a site admin with email "joy@smart-unlimited.com" and password "password". You'll need these details later to sign in. Skip this step if you don't want seed data.

```
docker-compose run web bundle exec rake db:seed
```

Then
```
docker-compose up
```

Those steps will take a little while as they download images and build
the docker containers.

When its stops spitting output to the console point your web browser at

http://localhost:3000

For development all mail sent out by Cuttlefish will actually go to mailcatcher.
To see the mailcatcher mail:

http://localhost:1080

To run the tests (do that from another window):
```
docker-compose exec web rake
```

## To install:

We use [Vagrant](https://www.vagrantup.com/) and [Ansible](http://docs.ansible.com/) to automatically set up a fresh server with everything you need to run Cuttlefish. It's a fairly complicated affair as Cuttlefish does have quite a few moving
parts but all of this is with the purpose of making it easier for the developer sending mail.

These instructions are specifically for installing the server at https://cuttlefish.oaf.org.au.

Currently the setup requires a relatively old version of Ansible (2.5.0) using Python 2.7.

### To install to a local test virtual machine

1. Create a file `~/.cuttlefish_ansible_vault_pass.txt` which contains the password for encrypting the secret values used in the deploy. The encrypted variables are at `provisioning/roles/cuttlefish-app/vars/main.yml`.

2. Download base box and build virtual machine with everything needed for Cuttlefish. This will take a while (at least 30 mins or so)
```
vagrant up
```

3. Deploy the application. As this is the first deploy it will take quite a while (5 mins or so). Further deploys will be much quicker. We're using the `--set-before local_deploy=true` flag to deploy to your local test virtual machine instead of production.
```
bundle exec cap --set-before local_deploy=true deploy:setup deploy:cold foreman:export foreman:start
```

4. Add to your local `/etc/hosts` file
```
127.0.0.1       cuttlefish.oaf.org.au
```

5. Point your web browser at https://cuttlefish.oaf.org.au:8443/

### To install on [Linode](https://www.linode.com/)

1. Login at the [Linode Manager](https://manager.linode.com/)

2. [Add a new Linode](https://manager.linode.com/linodes/add)

3. Select "Linode 8GB" at location "Fremont, CA"

4. Select your new Linode in the dashboard

5. Click "Deploy a Linux Distribution". Choose "Ubuntu 16.04 LTS" and choose a root password. Leave everything as default.

6. Click "Boot" and wait for it to start up

8. Update `provisioning/hosts` with the name of your server (e.g. li123-45.members.linode.com)

9. Create a file `~/.cuttlefish_ansible_vault_pass.txt` which contains the password for encrypting the secret values used in the deploy. The encrypted variables are at `provisioning/roles/cuttlefish-app/vars/main.yml`.

10. To provision the server for the first time you will need to supply the root password you chose in step 5. On subsequent deploys you won't need this. To supply this password edit the `./provision_production.sh` script and temporily add the `--ask-pass` argument to the last command, then run the script:

```
./provision_production.sh
```

11. Update the server name in `config/deploy.rb`

12. Deploy the application. As this is the first deploy it will take quite a while (5 mins or so). Further deploys will be much quicker
```
cap deploy:setup
cap deploy:cold
cap foreman:export
cap foreman:restart
```

13. At this stage you might want to snapshot the disk

14. Make sure that DNS for cuttlefish.oaf.org.au points to the server ip address

14. Point your browser at https://cuttlefish.org.au

At this point you should have a basic working setup. You should be able to send test mail and see it getting delivered.

Some further things to ensure things work smoothly

1. Add DNS TXT record for cuttlefish.oaf.org.au with "v=spf1 ip4:your.server.ip4.address ip6:your.server.ip6.address -all"

2. Set up incoming email for cuttlefish.oaf.org.au (In OpenAustralia Foundation's case using Google Apps for domain). Add addresses contact@cuttlefish.oaf.org.au, bounces@cuttlefish.oaf.org.au and sender@cuttlefish.oaf.org.au

2. Ensure that the devise email address is set to contact@cuttlefish.oaf.org.au

3. Set up reverse DNS. In the Linode Manager under "Remote Access" click "Reverse DNS" then for the hostname put in "cuttlefish.oaf.org.au" and follow the instructions. This step is necessary in order to be able to sign up to receive [Feedback loop emails](https://en.wikipedia.org/wiki/Feedback_loop_%28email%29).

## Deploying to production

One gotcha is that we're still on Capistrano 2 which doesn't apply database migrations
by default on deploys.

For normal deploys
```
cap deploy
```

To rollback a failed deploy
```
cap deploy:rollback
```

To deploy and run the migrations
```
cap deploy:migrations
```

## Screenshots
Done some development work which updates the look of the main pages? To update the screenshots
```
bundle exec rspec spec/features/screenshot_feature.rb
```
Then commit the results

## How to contribute

If you find what looks like a bug:

* Check the [GitHub issue tracker](http://github.com/mlandauer/cuttlefish/issues/)
  to see if anyone else has reported issue.
* If you don't see anything, create an issue with information on how to reproduce it.

If you want to contribute an enhancement or a fix:

* Fork the project on GitHub.
* Make your changes with tests.
* Commit the changes without making changes to any files that aren't related to your enhancement or fix.
* Send a pull request.
