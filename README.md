# Slack Notify Action

A GitHub Action that sends a richly-formatted Slack notification from a workflow — header, message, repo/branch context, and a "View Run" button — with minimal boilerplate.

Built by [Full Stack House](https://www.fullstackhouse.com) because we were copy-pasting the same Slack block payload across every repo.

## Why this exists

The [official `slackapi/slack-github-action`](https://github.com/slackapi/slack-github-action) is flexible but low-level — you hand-write the block payload every time. [`rtCamp/action-slack-notify`](https://github.com/rtCamp/action-slack-notify) is higher-level but webhook-only and has a fixed layout.

This action is a thin wrapper around `slackapi/slack-github-action` that:

- Supports **both bot tokens and incoming webhooks**.
- Produces a consistent layout: title header, mrkdwn body, `Repository` / `Branch` fields, and a "View Run" button.
- Adds a status emoji (✅ / ❌ / ℹ️) based on a simple `status` input.
- Prefixes the header (and mobile push notification text) with the repo name so context is visible at a glance.

## Usage

### With a Slack bot token (recommended — one token, many channels)

```yaml
- uses: fullstackhouse/slack-notify-action@v1
  with:
    bot-token: ${{ secrets.SLACK_BOT_TOKEN }}
    channel: "#alerts"
    title: "Production deploy failed"
    message: "Deploy of `api` failed on `main`."
    status: failure
```

### With an incoming webhook

```yaml
- uses: fullstackhouse/slack-notify-action@v1
  with:
    webhook-url: ${{ secrets.SLACK_WEBHOOK_URL }}
    title: "Nightly cleanup succeeded"
    message: "Removed 12 stale preview environments."
    status: success
```

### Fire on failure only

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh

  notify:
    if: failure()
    needs: [deploy]
    runs-on: ubuntu-latest
    steps:
      - uses: fullstackhouse/slack-notify-action@v1
        with:
          bot-token: ${{ secrets.SLACK_BOT_TOKEN }}
          channel: "#alerts"
          title: "🚨 Prod deploy failed"
          message: "Deploy failed on `${{ github.ref_name }}`."
          status: failure
```

## Inputs

| Name          | Required | Default | Description |
| ------------- | -------- | ------- | ----------- |
| `bot-token`   | one of   |         | Slack bot token (`xoxb-…`). Mutually exclusive with `webhook-url`. |
| `webhook-url` | one of   |         | Slack incoming webhook URL. Mutually exclusive with `bot-token`. |
| `channel`     | with `bot-token` | | Slack channel (e.g. `#alerts`). Required when using `bot-token`; ignored for `webhook-url`. |
| `title`       | yes      |         | Header text. |
| `message`     | yes      |         | Body (Slack mrkdwn). |
| `fields`      | no       |         | Optional JSON array of extra Slack field objects appended after the default Repository/Branch block, e.g. `[{"type":"mrkdwn","text":"*Region:*\nEU"}]`. |
| `status`      | no       | `info`  | `success` \| `failure` \| `info`. Picks the header emoji. |

The Slack message automatically includes the repository, branch, and a "View Run" button linking to the triggering workflow run — no extra inputs needed.

## Permissions

This action doesn't need any special `permissions:` block. It only needs the Slack credential you pass it.

## License

[MIT](./LICENSE)
