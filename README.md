# Docker containers for Drupal 7. Now with testing!

## Requirements

 - Install [ahoy](https://github.com/ahoy-cli/ahoy)

## Usage

 - Clone this repository
 - Clone your Drupal project as `docroot`
 -- `git clone git@yourre.po/project.git docroot`
 - The `docroot` directory is mounted in the web container
 - Run `docker-compose up`
 - Check out the available `ahoy` commands in `.ahoy.yml`
 - Enjoy your setup!
