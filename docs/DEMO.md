# 🎬 Demo Guide — Cloud Posture Scanner

This document explains how to demo the working system for evaluation.

---

## Demo Flow (5–7 minutes)

### 1. Show the Architecture (30 sec)
- Open `docs/architecture-diagram.svg` in a browser
- Walk through: Client → API Gateway → Lambda → AWS APIs → DynamoDB

### 2. Show the Source Code (1 min)
- Open the GitHub repository
- Point out:
  - `lambdas/run_cis_checks/lambda_function.py` — all 5 CIS checks in one file
  - `iac/iam_policy.json` — least-privilege IAM policy (no `*` on actions)
  - `frontend/index.html` — single-file dashboard, no build step

### 3. Show AWS Console (2 min)
Open these AWS Console tabs:
- **Lambda** → Show 4 functions with correct roles and timeouts
- **API Gateway** → Show `CloudPostureScannerAPI` → Stages → prod → Show invoke URL
- **DynamoDB** → `scan_results` table → Items tab (initially empty or previous scan)
- **IAM** → `CloudPostureScannerRole` → Show permissions (no `*` resources)

### 4. Live API Demo via Postman (1 min)

Run these in sequence:

```
GET  {invoke-url}/instances      → Shows EC2 instances
GET  {invoke-url}/buckets        → Shows S3 buckets with encryption/access status
GET  {invoke-url}/cis-results    → Shows stored results (may be empty first time)
POST {invoke-url}/scan           → Triggers live scan, returns PASS/FAIL results
GET  {invoke-url}/cis-results    → Now shows results stored in DynamoDB
```

### 5. Frontend Dashboard Demo (1 min)

1. Open the S3 static website URL
2. Enter the API Gateway URL in the input field
3. Click **Run Scan** — watch the scan execute live
4. Walk through the 3 tabs:
   - **EC2 Instances** — table with all fields
   - **S3 Buckets** — shows encryption and access icons
   - **CIS Benchmark** — shows PASS ✓ / FAIL ✗ with evidence and posture score

### 6. Show DynamoDB After Scan (30 sec)
- Go back to DynamoDB Console → `scan_results` table → **Explore items**
- Show stored records with `resource_id`, `status`, `evidence`, `timestamp`

---

## Expected CIS Results

In a typical free-tier AWS account you will see:

| Check | Likely Result | Reason |
|-------|--------------|--------|
| S3 Public Access | PASS | New buckets default to private |
| S3 Encryption | FAIL | Encryption not enabled by default on older buckets |
| Root MFA | FAIL | Often disabled on test accounts — fix this! |
| CloudTrail | FAIL | Not enabled by default on free tier |
| SG Open SSH/RDP | PASS/FAIL | Depends on any EC2 you have running |

> 💡 **Tip for demo**: Deliberately leave one S3 bucket without encryption and one without public access blocking so the FAIL cases are visible.

---

## Screenshots to Include in Submission

1. **Architecture diagram** (`docs/architecture-diagram.svg`)
2. **Lambda functions list** in AWS Console
3. **API Gateway endpoints** showing all 4 routes
4. **DynamoDB table items** after a scan run
5. **Dashboard - CIS tab** showing PASS/FAIL results with posture score
6. **Dashboard - S3 tab** showing encryption status

---

## Key Points to Highlight

- **Zero hardcoded credentials** — Lambda uses IAM execution role automatically
- **Serverless** — no servers to manage, scales to zero when not in use
- **Free Tier** — estimated $0/month cost for this project
- **Modular** — 4 separate Lambda functions, each with single responsibility
- **Extensible** — new CIS checks can be added to `run_cis_checks_lambda` without touching the API or frontend
