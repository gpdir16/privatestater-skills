# PrivateStater Claude Skills

Claude Code agent skills for integrating and managing [PrivateStater](https://privatestater.com) — Analytics, Captcha, and Feedback Widget.

## Available Skills

| Skill | Description |
|-------|-------------|
| `privatestater-install` | Install PrivateStater into any website (Analytics, Captcha, Feedback Widget) |
| `privatestater-openapi` | Query and manage PrivateStater data via the Open API |

## Installation

```bash
# Install the installation skill
npx skills add https://github.com/gpdir16/privatestater-skills --skill privatestater-install

# Install the Open API skill
npx skills add https://github.com/gpdir16/privatestater-skills --skill privatestater-openapi

# Install both
npx skills add https://github.com/gpdir16/privatestater-skills --skill privatestater-install --skill privatestater-openapi
```

## Skills

### `privatestater-install`

Helps you add PrivateStater to any website. Use this when you want to:

- Install the PrivateStater tracking script
- Add click tracking to buttons
- Implement Captcha on a form
- Enable the Feedback Widget

### `privatestater-openapi`

Helps you query and manage PrivateStater data via the Open API. Use this when you want to:

- Check visitor counts, page traffic, browser or device breakdowns
- View session duration and traffic heatmaps
- Read and manage feedback messages
- Check captcha success rates
- Automate settings via API
