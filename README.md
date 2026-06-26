# MPSTATS Data Gateway Skills

This directory contains installable agent skills for MPSTATS Data Gateway.

List available skills:

```bash
npx skills add ./agent-skills --list
```

Install the gateway skill locally:

```bash
npx skills add ./agent-skills --skill mpstats-gateway -g -a codex -a claude-code -y
```

After publishing this directory to GitHub, use the same package with a GitHub
URL:

```bash
npx skills add https://github.com/maximelziv-max/mpstats-gateway-skills --skill mpstats-gateway -g -a codex -a claude-code -y
```

Do not put internal API keys or MPSTATS master tokens in this repository.
