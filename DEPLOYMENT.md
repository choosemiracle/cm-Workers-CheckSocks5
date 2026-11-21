# choosemiracle CheckSocks5 Worker deployment

## 1. Create the Worker

In Cloudflare Dashboard, open **Workers & Pages** and create a Worker connected to this GitHub repository:

- Repository: `choosemiracle/cm-Workers-CheckSocks5`
- Production branch: `main`
- Build command: leave empty
- Deploy command: `npx wrangler deploy`
- Root directory: `/`

The repository includes `wrangler.toml` with entrypoint `_worker.js`.

## 2. Configure the Worker secret

In the Worker settings, open **Settings → Variables and Secrets** and add a secret:

- Name: `CHECK_TOKEN`
- Value: a strong random value generated locally (for example, `openssl rand -hex 32`)

Save and deploy the Worker. Do not put this value in GitHub files or commit it.

## 3. Verify the endpoint

The public landing page can load without authentication. The check endpoint must reject requests without the token:

```sh
curl -i "https://YOUR-WORKER.workers.dev/check?sstp=vpn:vpn@example.com:443"
```

Expected response without Authorization: HTTP 401.

Do not publish the token in a URL. The pipeline sends it in the Authorization header.

## 4. Configure the gate repository

In `choosemiracle/gate`, open **Settings → Secrets and variables → Actions** and add:

- `DOMAIN`: Worker hostname only, e.g. `check-sstp.your-subdomain.workers.dev` (no scheme, path or trailing slash)
- `CHECK_TOKEN`: exactly the same random value configured as the Worker secret

Then open **Settings → Pages** and ensure the source is **GitHub Actions**.

## 5. Run

Open **Actions → VPN Gate Node Check → Run workflow**. On success, the workflow publishes:

- `https://choosemiracle.github.io/gate/`
- `https://choosemiracle.github.io/gate/nodes.txt`

The workflow is scheduled every 30 minutes after the first successful setup.

## Security and interpretation

- The token protects the expensive proxy-check endpoint. The public dashboard and IP/resolve utilities remain public as in upstream.
- Do not share the token or commit it.
- VPN Gate SSTP endpoints are not guaranteed to be residential/home broadband. The dashboard classifies observed exit networks using IP intelligence; treat the classification as an estimate, not proof.
