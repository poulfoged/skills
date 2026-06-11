---
name: zebrasoft-ci-setup
description: Use when setting up GitLab CI/CD, deployment servers, syncthing, cloudflared, or gitlab-runner for Zebrasoft projects. Covers the full pipeline pattern: build on docker-github-runner, sync artifacts via syncthing, deploy on a dedicated Debian server with systemd.
---

# Zebrasoft CI/CD Setup

## Architecture

```
[docker-github-runner]                    [deploy server (e.g. therese-hansen)]
  Build job (tag: dotnet)                   Deploy job (tag: <app>-deploy)
  - npm install && npm run build            - unzip /home/deploy/<app>-{SHA}.zip
  - dotnet publish -> build/                - install systemd service
  - zip -> /home/deploy/                    - systemctl restart <app>
        |
        | syncthing (folder id: 4ex5h-qevd2, label: deploy)
        |
  /home/deploy/ <---------------------> /home/deploy/
```

## Known device IDs

| Server | Syncthing Device ID |
|---|---|
| `docker-github-runner` | `NCDPTT2-FA6GFPU-7SV6PA3-6YHKTDJ-SQPECLW-465YLWS-HBIDPUF-KQNEAAK` |
| `gitlab-runner` (second device on runner) | `DUZNG57-THR22BA-D4PYYDQ-SZZ577K-D6U645G-DWCK53S-KPCPH44-ASVQKA3` |
| `academy-test` | `FPKT3YC-UTPDEWD-6KTVGS6-6WCPEVE-2FXTILR-ALFR6O6-PMQLQNB-X4BAOA7` |
| `therese-hansen` | `SDYOLXI-4MTDYAS-L34RMAY-4VPWK2L-UGP675W-KGK5WXV-4RLJTDS-M44OWAR` |

`docker-github-runner` syncthing API key: `qNWq4dkGjGAx5Q44JLWdycmx3wiY5Zv9`
Syncthing deploy folder ID: `4ex5h-qevd2` (path `/home/deploy` on all servers)

## Repo files

### `.gitlab-ci.yml`

```yaml
stages:
  - build
  - deploy

build:
  stage: build
  tags:
    - dotnet
  script:
    - cd source/<AppFolder>
    - npm install
    - npm run build
    - cd ../..
    - dotnet publish source/<AppFolder>/<App>.csproj -c Release -o build
    - cp <app>.service build/
    - cd build
    - zip -r /home/deploy/<app>-${CI_COMMIT_SHORT_SHA}.zip *

deploy:
  stage: deploy
  when: manual
  tags:
    - <app>-deploy          # production: <app>-deploy  |  test: <app>-deploy-test
  script:
    - mkdir -p /home/sites/<app>
    - cd /home/sites/<app>
    - systemctl stop <app> || true
    - pkill -f "dotnet.*<App>.dll" || true
    - sleep 2
    - unzip -oq /home/deploy/<app>-${CI_COMMIT_SHORT_SHA}.zip
    - cp <app>.service /etc/systemd/system/<app>.service
    - systemctl daemon-reload
    - systemctl start <app>
```

**CRITICAL:** Use plain `systemctl` — NOT `systemctl --system`. On Debian
non-login shells `--system` causes "Failed to connect to bus" errors.

### `<app>.service`

```ini
[Unit]
Description=<AppName>

[Service]
WorkingDirectory=/home/sites/<app>
ExecStart=/usr/bin/dotnet /home/sites/<app>/<App>.dll
Restart=always
RestartSec=5
KillSignal=SIGINT
SyslogIdentifier=<app>
User=root
Environment=ASPNETCORE_ENVIRONMENT=Production

[Install]
WantedBy=multi-user.target
```

### `appsettings.json` — Elasticsearch endpoints

```json
{
  "Elasticsearch": {
    "Endpoints": [
      "http://172.22.22.10:9200",
      "http://172.22.22.11:9200"
    ]
  }
}
```

## Setting up a new Debian 11 deploy server

### 1. Install .NET 9 runtime

```bash
wget -q https://packages.microsoft.com/config/debian/11/packages-microsoft-prod.deb -O /tmp/ms.deb
dpkg -i /tmp/ms.deb
apt-get update -qq && apt-get install -y aspnetcore-runtime-9.0
```

### 2. Install syncthing (official repo — Debian's bundled version is too old)

```bash
wget -q -O /tmp/syncthing.asc https://syncthing.net/release-key.txt
gpg --dearmor < /tmp/syncthing.asc > /usr/share/keyrings/syncthing-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/syncthing-archive-keyring.gpg] https://apt.syncthing.net/ syncthing stable" \
  > /etc/apt/sources.list.d/syncthing.list
apt-get update -qq && apt-get install -y syncthing

# Create system user
useradd --system --home /var/lib/syncthing --create-home --shell /bin/false syncthing
chown -R syncthing:syncthing /var/lib/syncthing

# Enable and start
systemctl enable syncthing@syncthing
systemctl reset-failed syncthing@syncthing 2>/dev/null || true
systemctl start syncthing@syncthing

# Wait for config to generate, then open GUI to 0.0.0.0
sleep 5
sed -i 's|<address>127.0.0.1:8384</address>|<address>0.0.0.0:8384</address>|' \
  /var/lib/syncthing/.local/state/syncthing/config.xml
systemctl restart syncthing@syncthing
```

Get the new device ID and API key:

```bash
syncthing --device-id   # ignore any /root warning — the second line is the real ID
grep apikey /var/lib/syncthing/.local/state/syncthing/config.xml
```

Create the deploy folder and set ownership:

```bash
mkdir -p /home/deploy
touch /home/deploy/.stfolder
chown -R syncthing:syncthing /home/deploy
```

### 3. Wire syncthing — add new server to docker-github-runner

Run on docker-github-runner (or via its SSH):

```bash
APIKEY="qNWq4dkGjGAx5Q44JLWdycmx3wiY5Zv9"
NEW_ID="<new-server-device-id>"
NEW_NAME="<new-server-hostname>"

# Register the new device
curl -s -X PUT -H "X-API-Key: $APIKEY" \
  http://localhost:8384/rest/config/devices/$NEW_ID \
  -H "Content-Type: application/json" \
  -d "{\"deviceID\":\"$NEW_ID\",\"name\":\"$NEW_NAME\",\"addresses\":[\"dynamic\"],\"compression\":\"metadata\",\"introducer\":false,\"skipIntroductionRemovals\":false,\"autoAcceptFolders\":false}"

# Add it to the deploy folder
curl -s -H "X-API-Key: $APIKEY" http://localhost:8384/rest/config/folders/4ex5h-qevd2 | \
  python3 -c "
import sys, json
cfg = json.load(sys.stdin)
cfg['devices'].append({'deviceID':'$NEW_ID','introducedBy':'','encryptionPassword':''})
print(json.dumps(cfg))
" > /tmp/folder_cfg.json

curl -s -X PUT -H "X-API-Key: $APIKEY" \
  http://localhost:8384/rest/config/folders/4ex5h-qevd2 \
  -H "Content-Type: application/json" \
  -d @/tmp/folder_cfg.json
```

Then on the new server, add docker-github-runner and join the folder:

```bash
APIKEY="<new-server-api-key>"

curl -s -X PUT -H "X-API-Key: $APIKEY" \
  http://localhost:8384/rest/config/devices/NCDPTT2-FA6GFPU-7SV6PA3-6YHKTDJ-SQPECLW-465YLWS-HBIDPUF-KQNEAAK \
  -H "Content-Type: application/json" \
  -d '{"deviceID":"NCDPTT2-FA6GFPU-7SV6PA3-6YHKTDJ-SQPECLW-465YLWS-HBIDPUF-KQNEAAK","name":"docker-github-runner","addresses":["dynamic"],"compression":"metadata","introducer":false,"skipIntroductionRemovals":false,"autoAcceptFolders":false}'

curl -s -X PUT -H "X-API-Key: $APIKEY" \
  http://localhost:8384/rest/config/folders/4ex5h-qevd2 \
  -H "Content-Type: application/json" \
  -d '{
    "id": "4ex5h-qevd2",
    "label": "deploy",
    "path": "/home/deploy",
    "type": "sendreceive",
    "devices": [
      {"deviceID":"NCDPTT2-FA6GFPU-7SV6PA3-6YHKTDJ-SQPECLW-465YLWS-HBIDPUF-KQNEAAK","introducedBy":"","encryptionPassword":""},
      {"deviceID":"DUZNG57-THR22BA-D4PYYDQ-SZZ577K-D6U645G-DWCK53S-KPCPH44-ASVQKA3","introducedBy":"","encryptionPassword":""}
    ],
    "rescanIntervalS": 3600,
    "fsWatcherEnabled": true,
    "fsWatcherDelayS": 10,
    "ignorePerms": false,
    "autoNormalize": true
  }'
```

Check sync status:

```bash
curl -s -H "X-API-Key: $APIKEY" "http://localhost:8384/rest/db/status?folder=4ex5h-qevd2" | \
  python3 -c "import sys,json; d=json.load(sys.stdin); print('state:', d.get('state'), 'needFiles:', d.get('needFiles',0))"
```

### 4. Install cloudflared

```bash
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb \
  -o /tmp/cloudflared.deb
dpkg -i /tmp/cloudflared.deb
cloudflared service install <tunnel-token>
systemctl is-active cloudflared
```

### 5. Install and register gitlab-runner

```bash
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | bash
apt-get install -y gitlab-runner
```

Register (newer versions do NOT accept `--tag-list` — set tags in the GitLab UI instead):

```bash
gitlab-runner register \
  --url https://gitlab.com \
  --token glrt-<token-from-gitlab-ui> \
  --executor shell \
  --description "<server-name>" \
  --non-interactive
```

**CRITICAL:** The gitlab-runner systemd service hardcodes `--user gitlab-runner`.
Override it to run as root — otherwise `systemctl`, `cp` to `/etc/systemd/`, and
`unzip` to `/home/sites/` will all fail with permission errors:

```bash
mkdir -p /etc/systemd/system/gitlab-runner.service.d/
cat > /etc/systemd/system/gitlab-runner.service.d/override.conf << 'EOF'
[Service]
ExecStart=
ExecStart=/usr/bin/gitlab-runner "run" "--config" "/etc/gitlab-runner/config.toml" "--working-directory" "/home/gitlab-runner" "--service" "gitlab-runner" "--user" "root"
EOF
systemctl daemon-reload && systemctl restart gitlab-runner
```

Also set `user = "root"` inside the runner stanza in `/etc/gitlab-runner/config.toml`.

Create directories:

```bash
mkdir -p /home/sites/<app>
```

### 6. Fix git safe.directory (first deploy only)

The build directory is initially created by the `gitlab-runner` user before the
root override takes effect. Git will refuse to operate on it. Fix once:

```bash
git config --global --add safe.directory /home/gitlab-runner/builds/<runner-token-short>/0/<group>/<repo>
```

The exact path is shown in the error message in the failed job log.

## Known gotchas

| Problem | Cause | Fix |
|---|---|---|
| `systemctl: Failed to connect to bus` | `--system` flag on Debian non-login shell | Use plain `systemctl` (drop `--system`) |
| `cp: Permission denied` on `/etc/systemd/system/` | Runner not truly running as root | Add systemd drop-in override (step 5 above) |
| `unzip: Permission denied` on `/home/sites/<app>` | Same root cause | Same fix — root override |
| `fatal: detected dubious ownership` in git | Build dir owned by gitlab-runner, job runs as root | `git config --global --add safe.directory <path>` |
| `syncthing@syncthing: Failed to determine user credentials` | `syncthing` OS user not created before enabling service | `useradd --system syncthing` first, then `systemctl reset-failed && start` |
| Syncthing folder error: `mkdir .stfolder: permission denied` | `/home/deploy` owned by root, syncthing runs as `syncthing` | `chown -R syncthing:syncthing /home/deploy` |
| Syncthing installs ancient v1.12 | Debian 11 bundles old version | Use `apt.syncthing.net` repo with `gpg --dearmor` keyring method |
| `apt-key is deprecated` warning | Old keyring method | Use `gpg --dearmor` into `/usr/share/keyrings/` (see step 2) |
| `--tag-list` flag rejected by gitlab-runner register | Removed in recent versions | Set tags in GitLab UI: Settings → CI/CD → Runners |

## Runner tag naming convention

- Production server: `<app>-deploy`
- Test/staging server: `<app>-deploy-test`
