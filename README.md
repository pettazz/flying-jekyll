# flying-jekyll
A skeleton for setting up a Jekyll blog that automatically builds and deploys on fly.io via GitHub Actions

## Usage

### Set up Fly
1. Sign up for a Fly account at https://fly.io/app/sign-up
2. Install the `flyctl` tool: https://fly.io/docs/hands-on/install-flyctl/
3. Log into your new account by running `flyctl auth login`

### Setup Your App
1. Run `flyctl launch` to set up your app, answer as follows:
2. Answer `y` to copy the existing configuration to your new app
3. Choose an app name. This name is only used for Fly on your dashboard and your automatic dev url (example: https://flying-jekyll.fly.dev) and will not be visible to anyone else if using a custom domain name, so choose something memorable for yourself.
4. Choose a region. Select whichever region is probably closest to you or whoever you think will visit your site. You can always update this later if needed: https://fly.io/docs/reference/regions/
5. Note the "Admin URL" and "Hostname" given
6. Answer `N` (or press enter) to creating a Postgresql database
7. Answer `N` (or press enter) to creating a Redis database
8. Answer `y` to deploy now
9. Once it's completed successfully (see the message "--> v0 deployed successfully"), visit the hostname from step 4 to see the site up live. Unless you've changed anything, you should see the basic "Welcome to Fying Jekyll" page, identical to https://flying-jekyll.fly.dev.

### Setup Your Domain (Optional)
1. Run `flyctl ips list` to get your app's public IP
2. In your domain provider's DNS settings, add an A record pointing to your v4 IP and an AAAA record pointing to your v6 IP. This will depend on your particular domain provider, but likely the "Host" will be `@` and the "Value" will be the IP address for each record. Be sure to delete any other A records or CNAMEs from other configurations or parking pages.
3. Run `flyctl certs create example.com` replacing `example.com` with your actual domain.
4. Run `flyctl certs show example.com` to check the status. This typically only takes a few minutes. You may be asked for an additional ownership verification: if the Status is "Ready" and there is a "DNS Validation Instructions" field listed, you will need to complete as directed. This is typically adding a CNAME in your domain's DNS from the given hostname (something like `_acme-challenge.example.com`) to the target value (something like `example.com.5xzw.flydns.net`). Once completed, check again with `flyctl certs show example.com` to see the status; this should also only take a few minutes. When finished you will see a message like "The certificate for example.com has been issued."
5. Verify that visiting your domain brings you to your "Welcome to Flying Jekyll" page.

### Setup GitHub Actions
1. Run `flyctl auth token` to get an API token for your app
2. In your GitHub repository, click the Settings tab, and select Secrets and Variables > Actions from the left menu
3. Click the "New repository secret" button and fill in the Name `FLY_API_TOKEN` and the Secret with the API token from step 1, then click Add secret
4. Make some change to your site (like changing the text in `index.markdown`), and commit/push to your repository
5. You can see the build Action status in the Actions tab on your repository's page. If it fails you will receive an email with error details.
6. Once completed successfully, verify that you can see your changes on your site (either fly.io dev hostname or your custom domain)

#### Using Secrets 
The workflow and Dockerfile are configured to pass any Secrets from the GitHub settings into the build context as environment variables, and interpolate those values into `_config.yml` and `nginx.conf`. For example, if you set a repository secret called `MY_COOL_API_KEY`, you will be able to reference `${MY_COOL_API_KEY}` in your settings so as to not expose this key by checking it into the repository.

1. In `_config.yml` or `nginx.conf`, replace whatever sensitive value you are trying to hide with a variable name, i.e.:

  Before:

  ```yaml
  webservice:
    access_token: 1234-dead-beef
  ```

  After:

  ```yaml
  webservice:
    access_token: ${MY_COOL_API_KEY}
  ```

Any existing environment variables will be interpolated, so be careful if using Nginx variables. 

2. In your GitHub repository, click the Settings tab, and select Secrets and Variables > Actions from the left menu
3. Click the "New repository secret" button and fill in the Name of the variable you want to use (i.e., `MY_COOL_API_KEY`) and the Secret with its value (i.e., `1234-dead-beef`), then click Add secret
4. Build and enjoy!

## Optional Dark Visitors Support

[Dark Visitors](https://darkvisitors.com) is a service that provides live monitoring for automated User Agents and sources that information to maintain a list of known AI scrapers and crawlers. You can configure your Jekyll site to both send events to Dark Visitors for tracking and to block AI scrapers. It's free up to a certain amount of traffic.

1. Sign up for Dark Visitors and get an access token: https://darkvisitors.com/docs/analytics
2. Set your access token as a secret called DARK_VISITORS_TOKEN in GitHub settings (see [Using Secrets](#using-secrets) above)
3. Replace `nginx.conf` with `nginx-darkvisitors.conf`, either copy paste the contents or rename it to `nginx.conf` and replace the old one
4. In the `Dockerfile`, change the base image from plain Nginx to OpenResty

Replace the line:

    FROM nginx:alpine AS runner

with:

    FROM openresty/openresty:alpine AS runner

And update the destination of `nginx.conf`,

Replace the line:

    COPY --from=builder /build-zone/nginx.conf /etc/nginx/nginx.conf

with: 

    COPY --from=builder /build-zone/nginx.conf /etc/nginx/conf.d/site.conf

You will notice comments in the Dockerfile mentioning both of these.

5. Once your site is deployed, you can check the Dark Visitors realtime dashboard to verify that it's working. Visiting your site with a known AI user agent like `ClaudeBot` should result in a 403

## Enjoy all the indescribable pleasures of owning a website