# 2-Hour Presentation Runbook

This is a fast, practical script to present Cloud Mirror with real data and failover controls.

## Outcome You Can Show

- Local API serving live task data
- Cloud backup API available on Cloud Run
- Manual failover and recovery controls working
- Distributed lock status visible
- Replication and failover metrics visible

## 0. Time Budget

- 0 to 25 min: boot local app + verify cloud URL
- 25 to 55 min: seed real data + validate sync and mirror endpoints
- 55 to 85 min: dry-run demo script once
- 85 to 120 min: final run and screenshots/slides

## 1. Start local app

From project root:

```bash
cp .env.example .env
```

Edit .env and set at least:

- DB_HOST, DB_PORT, DB_NAME, DB_USER, DB_PASSWORD (local PostgreSQL)
- SYNC_ENABLED=true
- CLOUD_DB_HOST, CLOUD_DB_PORT, CLOUD_DB_NAME, CLOUD_DB_USER, CLOUD_DB_PASSWORD (cloud PostgreSQL)
- MIRROR_ENABLED=true
- MIRROR_NODE_ID=local-node
- MIRROR_LOCK_BUCKET=<your-gcs-lock-bucket>

Then run:

```bash
npm ci
npm start
```

## 2. Open local demo website

Open this in browser:

- http://localhost:3000/demo/

Use it to:

1. Check Health
2. Create Task
3. List Tasks
4. Trigger Failover
5. Trigger Recovery
6. Get Metrics
7. (Optional) Set Cloud API base URL and click Run Demo Sequence

## 3. Seed actual data quickly

```bash
chmod +x scripts/demo-seed.sh scripts/demo-dr-flow.sh
./scripts/demo-seed.sh http://localhost:3000 25
```

Verify:

```bash
curl -s http://localhost:3000/tasks | jq 'length'
curl -s http://localhost:3000/mirror/status | jq
curl -s http://localhost:3000/metrics
```

## 4. Run live failover flow (local + cloud)

You need cloud service URL from Cloud Run, for example:

- https://demo-local-node-xxxxx-uc.a.run.app

Run:

```bash
./scripts/demo-dr-flow.sh http://localhost:3000 https://YOUR_CLOUD_RUN_URL
```

This script performs:

1. Health checks local and cloud
2. Seeds local tasks
3. Triggers local failover
4. Shows mirror status/lock
5. Writes task in cloud during failover
6. Triggers recovery
7. Prints metrics

## 5. 7-minute talk track (suggested)

1. Problem statement:
   We need service continuity when local node fails.
2. Architecture:
   Local node is primary, cloud node is warm backup with lock coordination.
3. Live data:
   Show tasks created from local UI and API.
4. Failover:
   Trigger /mirror/failover and show local read-only + lock ownership.
5. Cloud continuity:
   Show cloud writes during failover.
6. Recovery:
   Trigger /mirror/recover and show normal mode restored.
7. Metrics:
   Show /metrics for processed events, lag, failover state.

## 6. Live click script (90 seconds)

On /demo page:

1. Set Primary API URL to `http://localhost:3000`
2. Set Cloud API URL to your Cloud Run URL
3. Click `Run Demo Sequence`
4. Narrate output sections: health, data creation, failover, recovery, metrics

## 7. If DNS routing is asked in Q&A

Current demo uses API-level failover controls and lock semantics.
DNS automation is represented by the runbook and can be added as a controller that updates Cloud DNS or Cloudflare records based on health thresholds.

For this presentation, demonstrate:

- consistent health state
- lock-based failover safety
- active node switch and recovery

These prove disaster-recovery behavior end to end.

## 8. Backup plan if cloud is flaky

If cloud URL is slow/unavailable near presentation time:

1. Present local demo UI at /demo
2. Run failover and recovery endpoints locally
3. Show metrics output and lock transitions
4. Use previously captured screenshots of cloud health/task responses

This still demonstrates architecture, failover logic, and observability.

## Deploy to GCP

PROJECT_ID='cloud-recovery-492721' \
DB_PASSWORD='postgres' \
