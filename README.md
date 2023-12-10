# end-to-end-tests
Full end-to-end integration tests for JLINC apps

## Setup

Install and start a postgres server.

Install and start a redis server.


```
$ ./scripts/initialize
```

Create `./.env`

```
DATABASE_HOST=postgresql://localhost
REDIS_HOST=redis://localhost:6379
HEADLESS=1
```

## Run the servers in development

### Setup

Install and setup [puma-dev](https://github.com/puma/puma-dev)

Create puma-dev server port files:

```
echo 3030 > ~/.puma-dev/did.jlinc
echo 3040 > ~/.puma-dev/assets.jlinc
echo 3050 > ~/.puma-dev/devhooks.jlinc.jlinc
echo 3011 > ~/.puma-dev/b-api.jlinc
echo 3021 > ~/.puma-dev/b-portal.jlinc
echo 3020 > ~/.puma-dev/a-api.jlinc
echo 3010 > ~/.puma-dev/a-portal.jlinc
echo 3666 > ~/.puma-dev/styleguide.jlinc
```

Create `./servers/development.config.js`

```
const fs = require('fs')

const readPumaDevPortFile  = name =>
  fs.readFileSync(process.env.HOME+'/.puma-dev/'+name).toString().trim()

const scheme = 'http' // 'https'

const DID_SERVER_PORT           = readPumaDevPortFile('did.jlinc')
const ASSETS_SERVER_PORT        = readPumaDevPortFile('assets.jlinc')
const A_API_PORT                = readPumaDevPortFile('a-api.jlinc')
const B_API_PORT                = readPumaDevPortFile('b-api.jlinc')
const B_PORTAL_PORT             = readPumaDevPortFile('b-portal.jlinc')
const A_PORTAL_PORT             = readPumaDevPortFile('a-portal.jlinc')
const STYLEGUIDE_PORT           = readPumaDevPortFile('styleguide.jlinc')

module.exports = {
  LOG_LEVEL:                  'silly',

  DATABASE_HOST:              process.env.DATABASE_HOST,
  REDIS_HOST:                 process.env.REDIS_HOST,

  ASSETS_SERVER_SECRET:       GET_THIS_FROM_ANOTHER_DEVELOPER,
  ASSETS_SERVER_STORAGE_PATH: './tmp/assets',
  STRIPE_KEY:                 GET_THIS_FROM_ANOTHER_DEVELOPER,
  STRIPE_SECRET:              GET_THIS_FROM_ANOTHER_DEVELOPER,
  POSTMARK_KEY:               GET_THIS_FROM_ANOTHER_DEVELOPER,
  TWILIO_ACCOUNT_SID:         GET_THIS_FROM_ANOTHER_DEVELOPER,
  TWILIO_AUTH_TOKEN:          GET_THIS_FROM_ANOTHER_DEVELOPER,
  TWILIO_FROM_PHONE_NUMBER:   GET_THIS_FROM_ANOTHER_DEVELOPER,

  DID_SERVER_PORT,
  ASSETS_SERVER_PORT,
  A_API_PORT,
  B_API_PORT,
  STYLEGUIDE_PORT,
  B_PORTAL_PORT,
  A_PORTAL_PORT,

  DID_SERVER_URL:           `${scheme}://did.jlinc.test`,
  ASSETS_SERVER_URL:        `${scheme}://assets.jlinc.test`,
  A_API_URL:                `${scheme}://a-api.jlinc.test`,
  B_API_URL:                `${scheme}://b-api.jlinc.test`,
  STYLEGUIDE_URL:           `${scheme}://styleguide.jlinc.test`,
  B_PORTAL_URL:             `${scheme}://b-portal.jlinc.test`,
  A_PORTAL_URL:             `${scheme}://a-portal.jlinc.test`,

  // DID_SERVER_URL:           `${scheme}://localhost:${DID_SERVER_PORT}`,
  // ASSETS_SERVER_URL:        `${scheme}://localhost:${ASSETS_SERVER_PORT}`,
  // A_API_URL:                `${scheme}://localhost:${A_API_PORT}`,
  // B_API_URL:                `${scheme}://localhost:${B_API_PORT}`,
  // B_PORTAL_URL:             `${scheme}://localhost:${B_PORTAL_PORT}`,
  // STYLEGUIDE_URL:           `${scheme}://localhost:${STYLEGUIDE_PORT}`,
  // A_PORTAL_URL:    `${scheme}://localhost:${A_PORTAL_PORT}`,

  A_API_DATABASE:             'development_a_api',
  B_API_DATABASE:             'development_b_api',
  DID_DATABASE:               'development_did',

  A_API_REDIS_DB: 1,
  B_API_REDIS_DB: 2,

  A_API_WRITE_EMAILS_TO:       './tmp/development.emails',
  B_API_WRITE_EMAILS_TO:       './tmp/development.emails',
  A_API_WRITE_SMS_MESSAGES_TO: './tmp/development.sms-messages',
}
```

### Starting the servers


run `./scripts/development --help`

run `./scripts/development -rf` to reset and re-fixture your development environment

run `./scripts/development` to just start the servers

#### Logging

For verbose logging:

```bash
LOG_TO_CONSOLE=1 ./scripts/development [options]
```

### using the scripts within each submodule server

The scripts inside of each server all rely on the environment variables within `.development.env`.

You can see each config file for each server by running `./scripts/development --env --dont-start`

You can write each config to the `.development.env` file with each server by
running `./scripts/development -W --dont-start` or `./scripts/development -WD`

This will allow you to run, for example: `./scripts/db-console` withing the `a-api` and connect
to the same database as your running development environment.


## Run the tests

run `npm test` and it should do everything you need.

checkout `cat ./scripts/test`.

If you want to avoid the slow portals build step you can
run `./scripts/buildPortals` manually and then
run `mocha ./test/the-one-youre-working-on.spec.js`
and it will re-use the build portals, which should save
you like 5-30 seconds each time you run mocha.

## Steps to run locally

Steps:

1) install node 18.13.0
2) go to `new_sodium_and_new_node` branch   <---- wil help installing at node version 18.13.0
3) install node modules using `./scripts/initialize` <---> do this every time you change branch
4) add env vars to `./servers/development.config.js` <---> will upload the file
5) install REDIS, if you are an ubuntu user, it is recommended to install it from official website using the option of 'install on ubuntu' not 'from SnapCraft' to avoid automatic running for redis which may cause errors when running the app
6) add env vars to .env i.e :
    REDIS_HOST=redis://localhost:6379
    HEADLESS=0
    LOG_LEVEL=debug
    LOG_TO_CONSOLE=1
7) install postgres v14 using this command ./scripts/postgresql-docker-image start, if you are ubuntu user you may get permission error so you have to run it, so try these commands after installation to get permission
   1) `sudo -i` to get root permission
8) add postgres env vars to .env i.e : DATABASE_HOST=postgresql://postgres:postgres@localhost:5432
--> then copy .test.env at a-api and b-api to be .development.env and replace them with there values
9) apply migrations using this command cd a-api && npm run db:setup, do the same for b-api by using cd b-api && npm run db:setup and finally do the same for did_development and return back to b-api and set its $DATABASE_URL
and same for the did server but it's a bash script so you have to run it inside docker or normally if you don't use docker
10) install puma/puma/puma ---> this one to make you run localhost with ssh to make it work at the domain .test so the frontend domain will be [https://a-portal.jlinc.test/login](https://a-portal.jlinc.test/login)
    Install and setup [puma-dev](https://github.com/puma/puma-dev)
    Prerequisites:
        You should have Ruby and RubyGems installed on your Ubuntu system.
          Step 1: Install Ruby and RubyGems
          If you don't have Ruby and RubyGems installed, you can do so by following these steps:
          # Update the package list
            ```bash sudo apt update```
          # Install Ruby and RubyGems
            ```sudo apt install ruby-full```
          Verify the Ruby and RubyGems installation by running the following commands:
          ```bash
          ruby -v
          gem -v```
          Step 2: Install Puma-dev
          You can install Puma-dev using RubyGems, which is the standard package manager for Ruby:
          ```bash gem install puma-dev```
    Create puma-dev server port files:

    ```bash
    echo 3030 > ~/.puma-dev/did.jlinc
    echo 3040 > ~/.puma-dev/assets.jlinc
    echo 3050 > ~/.puma-dev/devhooks.jlinc.jlinc
    echo 3011 > ~/.puma-dev/b-api.jlinc
    echo 3021 > ~/.puma-dev/b-portal.jlinc
    echo 3020 > ~/.puma-dev/a-api.jlinc
    echo 3010 > ~/.puma-dev/a-portal.jlinc
    echo 3666 > ~/.puma-dev/styleguide.jlinc
    echo 3070 > ~/.puma-dev/tru
    echo 3080 > ~/.puma-dev/tru-portal
    echo 3045 > ~/.puma-dev/devhooks.jlinc
    ```

    for ubuntu users only you have to install dev-tld-resolver [A Simple top level domain resolver for linux based development environment.] from <https://github.com/puma/dev-tld-resolver>
11) Additional steps for ubuntu users:
    1) Set Up DNS
        Type the following command to open the hosts file with administrative privileges:
          `sudo nano /etc/hosts`
          You will be prompted to enter your password.
            In the text editor that opens, you will see the current entries in the hosts file. Replace the existing entries with the following lines:
            `
            127.0.0.1 b-portal.jlinc.test
            127.0.0.1 a-api.jlinc.test
            127.0.0.1 a-portal.jlinc.test
            127.0.0.1 styleguide.jlinc.test
            127.0.0.1 tru.test
            127.0.0.1 assets.jlinc.test
            127.0.0.1 tru-portal.test
            127.0.0.1 devhooks.jlinc.test`
            Press Ctrl+O to save the file, and then press Enter to confirm the filename.
            Press Ctrl+X to exit the text editor.
            Flush the DNS cache on your system to ensure the changes take effect. In the terminal, enter the following command:
            `sudo service network-manager restart`
            SSL Setup
              Copy the SSL certificates to your desktop by running the following commands:
              cp ~/.puma-dev-ssl/cert.pem ~/
              cp ~/.puma-dev-ssl/key.pem ~/
            This will copy the cert.pem and key.pem files at home.
            Open Your browser and search for the settings to certificates:
            In the "Certificates" settings, click on the "Authorities" tab, then click on the "Import" button. A file browser window will open.
            Navigate to your desktop using the file browser and select the cert.pem file that you copied earlier. Then click "Open" to import the certificate [cert.pem, key.pem] from home.

12) We need to create User Data Server by this command cd ./b-api && ./scripts/createUserDataServer pubkey secret-usd url .... the pubkey and secret and url are from the ./servers/development.config.js file at a-api section
13) Run both commands `redis-server` to run redis server and `puma-dev` to run puma but if you are an Ubuntu user run `puma-dev -sysbind` instead of `puma-dev`
14) run again using LOG_TO_CONSOLE=1 ./scripts/development `to see errors
15) you should see "READY!" at the end of the logs

## changing user's password

1)find your enc_password

SELECT enc_password FROM users WHERE username='ahmed';

result = $2b$12$MnN32SGRu9Ho0QISlCmzT.VuPMKaO0e7/l0uhIWWwS7klQOwaFYAe

2)update other person password using his username

UPDATE users SET enc_password = '$2b$12$MnN32SGRu9Ho0QISlCmzT.VuPMKaO0e7/l0uhIWWwS7klQOwaFYAe' WHERE username = 'TruVA';



