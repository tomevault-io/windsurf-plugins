---
trigger: always_on
description: This will be the standard operating procedure. When I say, "Do the thing." or "build my bonsai"– this is what I want you to do.
---

This will be the standard operating procedure. When I say, "Do the thing." or "build my bonsai"– this is what I want you to do.

Here's what it's going to do:

We are working with the latest version of Radicle, a Roots WP project. I need you to get it up and running on a Lima VM (using vm-start.sh), which will run `trellis vm start` and configure the port in my ssh config so that we can use the @development alias without extra steps.

Before you proceed, if I haven't provided them already, ask me for my 
[project_name] and [sitename].

The [project_name] will be the name of our project. 

The [sitename] is your desired URL, and will be used in the wp-cli.yml config such as example.com.

Provided [sitename] and [projectname]:
[sitename]=`chosen`
[projectname]=`chosen.bonsai.so`

Before you do anything, you will need to look at the project structure to determine if we are using the Roots open-source stack (Sage 11, Bedrock, and Trellis), or whether we're using Radicle, their paid project which combines all of those into a single cleaner project structure.

If we're using the open-source Roots stack, you will see a completely empty project other than a .cursorrules file. 

If we're using Radicle, the project will look like this:
|–– bedrock
|–– config
|–– database
|–– public
|–– resources
|–– routes
|–– scripts
|–– storage
|–– tests

If we are using Radicle, then Trellis will be missing. That is one of the first thing's we'll install. Always use the latest release of a project or tool if unspecified.

– Check to make sure the /trellis folder doesn't already exist, if not then run `php .radicle-setup/trellis.php`
– `cd trellis && trellis init` (This will prompt me "What domain name should Trellis be configured for? (e.g. example.com):" Use the [sitename] for this. Wait until this is assigned before moving on to the next step.)
– `trellis vm start` - this should provision the development server. If it doesn't run `trellis provision development`
– Align Lima VM with /etc/hosts and SSH Config (instructions below)
– `npm install && npm run build`
– `composer install`
– `composer config repositories.bonsai-cli vcs git@github.com:jackalopelabs/bonsai-cli.git`
– `composer require jackalopelabs/bonsai-cli:dev-kanban`
– Configure wp-cli.yml to use the @development, @staging, and @production aliases (found below)
– `wp @development acorn optimize:clear`
– `wp @development acorn bonsai:init` (bonsai.sh is then added to the project, and it should already be executable)
– `./scripts/bonsai.sh acorn bonsai:generate chosen`

If we're not using Radicle, and the project is empty (other than this file)– this will be your operating procedure, which is different than the Radicle process.

- Install Trellis and Bedrock: `trellis new [sitename]` (If this command isn't found, try running– `brew install roots/tap/trellis-cli`)
- Move the contents (site + app) of [projectname]/[sitename] into the project root: `mv chosen.bonsai.so/site chosen.bonsai.so/trellis . && rmdir chosen.bonsai.so`
- Install Sage 11: `cd site/web/app/themes && composer create-project roots/sage`
- Build theme assets: `cd sage && npm install && npm run build`
– Install alpine.js `npm install alpinejs --save`
- Go to Trellis and run: `trellis init`
- Install Galaxy Roles: `cd trellis && trellis galaxy install`
- Install Trellis: `trellis vm start`
- Align Lima VM with /etc/hosts and SSH Config (instructions below)
- Configure `site/wp-cli.yml` to use the @development, @staging, and @production aliases
- Activate Sage 11: `wp @development theme activate sage`
- Go to the `site` directory and config Bonsai CLI: `composer config repositories.bonsai-cli vcs git@github.com:jackalopelabs/bonsai-cli.git`
- Now require it: `composer require jackalopelabs/bonsai-cli:dev-kanban`
– `wp @development acorn optimize:clear`
– `wp @development acorn bonsai:init` (bonsai.sh is then added to the project, and it should already be executable)
– `./web/app/themes/sage/scripts/bonsai.sh acorn bonsai:generate chosen`

This is how you configure `site/wp-cli.yml` to use the @development, @staging, and @production aliases:

Once completed, test to see if the `@development` alias is working (`ssh [sitename]`). If it's not, then you might refer back to making sure that `etc/hosts` and `.ssh/config` are aligned with the new Lima port

```yaml
@development:
  ssh: [sitename-tld.test]/srv/www/[sitename]/current
@staging:
  ssh: web@staging.[sitename]/srv/www/[sitename]/current
@production:
  ssh: web@[sitename]/srv/www/[sitename]/current
```

Notice that on @development the [sitename-tld.test] will need to remove the desired TLD, so that it can use .test instead. That way we won't have 2 TLDs such as `.com.test`, `.so.test`, or `.io.test` etc.

If I were working on my project called Chosen with a [sitename]=`chosen.bonsai.so`, then you might update wp-cli.yml to look something like this:

```yaml
@development:
  ssh: chosen.bonsai.test/srv/www/chosen.bonsai.so/current
@staging:
  ssh: web@staging.chosen.bonsai.so/srv/www/chosen.bonsai.so/current
@production:
  ssh: web@chosen.bonsai.so/srv/www/chosen.bonsai.so/current
```

Configuring Trellis for deployment:

On Digital Ocean:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jackalopelabs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
