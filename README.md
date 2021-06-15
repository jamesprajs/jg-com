JeffGeerling.com Drupal Codebase
CI

This is the Drupal codebase that powers JeffGeerling.com.

The building of this project and the migration of JeffGeerling.com from Drupal 7 to Drupal 8 has been live-streamed on geerlingguy's YouTube channel; you can watch all the episodes and see episode summaries and resources here: Migrating JeffGeerling.com from Drupal 7 to Drupal 8 - How-to video series.

I decided to open-source my website's codebase to help other Drupal users see how I built and maintain this site. If you like what you see or have been helped in any way by this project, please consider supporting me via Patreon, GitHub Sponsors, or another affiliate link.

Deploying to Production
Currently the process for deploying runs from the Midwestern Mac infrastructure playbook:

ansible-playbook playbook.yml --tags=deploy
Local Environment
The first time you start using this project, you need to create your local settings file:

cp web/sites/default/example.settings.local.php web/sites/default/settings.local.php
If you have PHP and Composer installed locally, you can install project requirements with:

composer install
Otherwise, you can run this command inside the built Docker container using docker-compose exec drupal composer install after you run the next command to bring up the Docker environment.

Make sure you have Docker installed, then run the following command (in the same directory as this README file):

docker-compose up -d
Visit http://localhost/ to see the Drupal installation. Visit http://localhost:8025/ to see MailHog.

Note: If you're not running on Linux, this environment runs best with Docker for Mac's Mutagen caching enabled. Before starting the environment the first time, go into the Docker dashboard and enable caching for the directory where this repository is located. See Mutagen-based caching docs for instructions.

Installing Drupal
You can install Drupal using the install wizard, but we like to use Drush for more automation:

docker-compose exec drupal bash -c 'drush site:install minimal --db-url="mysql://drupal:$DRUPAL_DATABASE_PASSWORD@$DRUPAL_DATABASE_HOST/drupal" --site-name="Jeff Geerling" --existing-config -y'
Syncing the Database from Production
At some point, I'll write up how to do it all with Drush, automated.

For now:

Open Sequel Pro, download database from prod server.
Open Sequel Pro, upload database to local environment.
Updating Configuration
Any time configuration is changed or any modules or Drupal is upgraded, you should export the site's configuration using the command:

docker-compose exec drupal bash -c 'drush config:export -y'
And then push any changes to the Git repository before deploying the latest code to the site.

Upgrading Core (and Contrib)
Set up the site like normal, make sure it's installed.
Run composer update (to update everything).
Run docker-compose exec drupal bash -c 'drush updb -y'
Run docker-compose exec drupal bash -c 'drush config:export -y'
Commit any changes and push to remote.
Run the deploy playbook to update the live site.
Linting PHP code against Drupal's Coding Standards
You can test the custom code in this project using phpcs:

docker-compose exec drupal bash -c './vendor/bin/phpcs \
  --standard="Drupal,DrupalPractice" -n \
  --extensions="php,module,inc,install,test,profile,theme" \
  web/themes/jeffgeerling \
  web/modules/custom'
Email Debugging with MailHog
The Docker configuration for this project enables a MailHog container, which has a web UI available at http://127.0.0.1:8025.

The php.ini file for the local environment is automatically configured to use mhsendmail to send PHP's email through the mailhog instance when you're using this project's Dockerfile to build the Drupal environment.

When Drupal sends an email, it should be visible in Mailhog's UI.