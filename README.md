# Auth0 Trunk-based Deployments

This is an example of using the [auth0-deploy-cli](https://github.com/auth0/auth0-deploy-cli) for deploying multiple environments using a trunk based approach.

In this example I have two environments, staging & production, which are configured using the JSON files in [the environment_configs directory](./environment_configs/).

Any secrets are provided using Github Secrets, and the deployment actions make use of Github Deployments/Environments to restrict secrets to deployment branches as well as requiring deployment approvals.

## How do I recreate this for my Auth0 tenant?
You need these:
- `./.github`
- `./environment_configs`
- `yarn.lock`
- `package.json`
- `.gitignore`
- `LICENSE`

Update `./environment_configs` to reference your tenants. 
I recommend setting `AUTH0_ALLOW_DELETE` to `false` until you're confident your setup is correctly captured in version control. If you set it to `true` then it **will delete any resources which exist in Auth0 but do not exist in your exported config**.


Once the configs are set up you will need to download your existing config from auth0.
You'll need a client ID and client secret to be set to run this command, Auth0 have details on setting this up for Github.
In short... New Application -> Machine-to-Machine -> All Permissions for Auth0 Management API.
Then you can copy the ID and Secret and set them as environment variables before running `yarn download:staging` (you can see the command in `package.json`).

Now your repository will look more like this one, with all the resources that you are using in Auth0.

## Deployment Triggers
 - All merges into `main` are deployed to the staging environment.
 - Staging deployments can be triggered manually on the `main` branch (useful in case of Github Actions outage or network issue)
 - Production deployments are triggered manually from the Actions tab


## Auth0 Traps & Pitfalls
### Blindly modifying JSON
I strongly recommend having an environment for manual testing. 
Changes should be made via the Auth0 Dashboard and then exported to JSON, the Yarn scripts in this repo can help with that.
The reason I recommend this is due to the way that Auth0 sometimes sets sensible defaults, or provides fields which can't actually be modified despite existing in the JSON specification.
It's better to catch these early by interacting directly with their API on the dashboard.
**Warning** when you download the config from Auth0 it will have templated in any keyword mappings, so be sure not to commit these, instead commit your original template strings.

### Action drift
This is when your find yourself needing different actions in different tenants.
For example perhaps you want to add an Allowlist to your testing tenant, to ensure that only employees can access it.
It's tempting to try to deploy the action to only one tenant, but this will result in differing architecture between environments, which is always best avoided.
There's also a field named `deployed`, but this is not useful to you in this case.
Currently the best workaround that I have is to deploy the Action to all environments, but have it check for the presence of a `secret`, and only run when that secret is defined.