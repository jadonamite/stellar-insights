# Email Digest System Integration Guide

## Overview
Automated weekly and monthly email reports with corridor and anchor performance summaries.

## Features
- Weekly digests (Monday 9 AM UTC)
- Monthly digests (1st of month 9 AM UTC)
- Top 10 corridors by volume
- Anchor performance metrics
- HTML formatted emails
- Manual trigger via API

## Environment Variables

Add to `.env`:
```bash
SMTP_HOST=smtp.gmail.com
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
DIGEST_RECIPIENTS=user1@example.com,user2@example.com
```

## Backend Integration

### 1. Add to Cargo.toml:
```toml
lettre = "0.11"
```

### 2. Add to main.rs:
```rust
mod email;

use email::{EmailService, DigestScheduler};

// Setup email service
let email_service = Arc::new(EmailService::new(
    env::var("SMTP_HOST")?,
    env::var("SMTP_USER")?,
    env::var("SMTP_PASS")?,
));

// Setup digest scheduler
let recipients: Vec<String> = env::var("DIGEST_RECIPIENTS")?
    .split(',')
    .map(|s| s.to_string())
    .collect();

let scheduler = Arc::new(DigestScheduler::new(
    email_service,
    cache.clone(),
    rpc_client.clone(),
    recipients,
));

// Start scheduler
tokio::spawn(async move { scheduler.start().await });

// Add manual trigger endpoint
.route("/api/digest/send", post(api::digest::send_digest_manual))
.with_state(scheduler)
```

## API Endpoint

### POST /api/digest/send
Manually trigger digest email.

**Request:**
```json
{
  "period": "Weekly",
  "recipients": ["user@example.com"]
}
```

**Response:**
```json
{
  "success": true,
  "message": "Digest sent to 1 recipients"
}
```

## Email Content

### Weekly/Monthly Report Includes:
- **Overview**: Total volume, average success rate
- **Top Corridors**: Success rate, volume, latency, change %
- **Top Anchors**: Success rate, transactions, volume

## Schedule

- **Weekly**: Every Monday at 9:00 AM UTC
- **Monthly**: 1st day of month at 9:00 AM UTC

## SMTP Configuration

### Gmail:
1. Enable 2FA
2. Generate App Password
3. Use `smtp.gmail.com` as host

### SendGrid:
1. Get API key
2. Use `smtp.sendgrid.net` as host
3. Use `apikey` as username

### AWS SES:
1. Verify domain
2. Get SMTP credentials
3. Use regional endpoint

## Testing

```bash
curl -X POST http://localhost:8080/api/digest/send \
  -H "Content-Type: application/json" \
  -d '{"period":"Weekly","recipients":["test@example.com"]}'
```
