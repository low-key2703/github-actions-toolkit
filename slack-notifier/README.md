# Slack Notifier Action

Send formatted notifications to Slack with status colors and user mentions.

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `webhook-url` | ✅ Yes | - | Slack webhook URL |
| `message` | ✅ Yes | - | Message content to send |
| `status` | ❌ No | `info` | Status type: `success`, `failure`, `warning`, `info` |
| `title` | ❌ No | `''` | Message title/heading |
| `footer` | ❌ No | `GitHub Actions` | Footer text |
| `mentions` | ❌ No | `''` | Mentions (e.g., `<!channel>`, `<!here>`, `<@USER_ID>`) |

## Status Colors

- `success` → Green
- `failure` → Red  
- `warning` → Yellow
- `info` → Gray

## Usage Examples

### Basic Usage
```yaml
- name: Notify Slack
  uses: ./slack-notifier
  with:
    webhook-url: ${{ secrets.SLACK_WEBHOOK }}
    message: 'Deployment completed successfully!'
```

### With Status and Title
```yaml
- name: Notify Success
  uses: ./slack-notifier
  with:
    webhook-url: ${{ secrets.SLACK_WEBHOOK }}
    message: 'Production deployment finished'
    status: 'success'
    title: '✅ Deploy Success'
```

### With Mentions
```yaml
- name: Notify Failure
  uses: ./slack-notifier
  with:
    webhook-url: ${{ secrets.SLACK_WEBHOOK }}
    message: 'Build failed! Please investigate.'
    status: 'failure'
    title: '❌ Build Failed'
    mentions: '<!channel>'
```

### Complete Example
```yaml
- name: Deployment Notification
  uses: ./slack-notifier
  with:
    webhook-url: ${{ secrets.SLACK_WEBHOOK }}
    message: 'Version 1.2.3 deployed to production'
    status: 'success'
    title: 'Production Deploy'
    footer: 'Deployed by ${{ github.actor }}'
    mentions: '<!here>'
```

## Setting Up Slack Webhook

1. Go to https://api.slack.com/apps
2. Create a new app → "From scratch"
3. Enable "Incoming Webhooks"
4. Add webhook to your channel
5. Copy the webhook URL
6. Add to GitHub Secrets as `SLACK_WEBHOOK`

## Tips

- Use `<!channel>` to notify everyone
- Use `<!here>` to notify active users only
- Find user IDs: Slack → Profile → More → Copy member ID
