# minalinsky
Crosspost from Twitter to Mastodon (based on Nitter)

## Notice
* Twitter crawling is based on Nitter, so only public accounts are supported.
* Supported tweet types
- [x] Text
- [x] Retweet
- [x] Image
- [ ] Replies to your own Tweet
- [ ] Video

## Usage
The easiest way to use this application is to run it as a docker compose service on your NAS or a VPS.

### Crosspost to Mastodon
You need to create a developer application on your Mastodon instance first

1. Go to `https://<your-mastodon-instance>/settings/applications/new`
2. On the form, put `minalinsky` for `Application name`
3. Under `Scopes`, uncheck `read`, `write` and `follow` checkboxes. Check `write:media` and `write:statuses` checkboxes.
4. Click `Submit`
5. Go to `https://<your-mastodon-instance>/settings/applications` and click the `minalinsky` application you just created
6. You will need the following
    * `Client key`
    * `Client secret`
    * `Your access token`

It's also strongly advised to obtain a burner Twitter account in order to crawl your own Twitter account. Do not enable 2FA for your burner Twitter account.

After obtaining the credentials, you can use the following `docker-compose.yml` to run the application
```yaml
services:
  nitter:
    image: ghcr.io/sekai-soft/nitter-self-contained
    container_name: nitter
    volumes:
      - nitter:/nitter-data
    environment:
      TWITTER_USERNAME: <ENTER YOUR BURNER TWITTER USERNAME HERE>
      TWITTER_PASSWORD: <ENTER YOUR BURNER TWITTER PASSWORD HERE>
      NITTER_ACCOUNTS_FILE: /nitter-data/guest_accounts.json
      DISABLE_NGINX: "1"
      INSTANCE_HOSTNAME: nitter
      INSTANCE_PORT: "80"
    restart: unless-stopped
    healthcheck:
      test: wget -nv --tries=1 --spider http://127.0.0.1:80 || exit 1
      interval: 5s
      timeout: 5s
      retries: 12
  minalinsky:
    image: ghcr.io/sekai-soft/minalinsky:latest
    volumes:
      - minalinsky:/app/dbs
#      - ./post.sh:/app/post.sh  # you can optionally mount a shell script at /app/post.sh to run after every Nitter crawl to perform tasks such as sending a heartbeat
    environment:
      SQLITE_FILE: /app/dbs/db.db
      NITTER_HOST: nitter
      NITTER_HTTPS: 'false'
      TWITTER_HANDLE: <REPLACE WITH YOUR TWITTER USERNAME, WITHOUT @>
      MASTODON_HOST: <REPLACE WITH YOUR MASTODON INSTANCE, e.g. mastodon.ktachibana.party>
      MASTODON_CLIENT_ID: <REPLACE WITH YOUR MASTODON CLIENT KEY>
      MASTODON_CLIENT_SECRET: <REPLACE WITH YOUR MASTODON CLIENT SECRET>
      MASTODON_ACCESS_TOKEN: <REPLACE WITH YOUR MASTODON ACCESS TOKEN>
      MASTODON_STATUS_LIMIT: '10'  # set the maximum of statuses to be posted at once
    restart: unless-stopped
volumes:
  nitter:
  minalinsky:
```

This will crawl your Twitter account every 5 minutes, and crosspost the tweets to your Mastodon account.

## Development

### Run unit tests
```shell
python -m unittest discover
```
