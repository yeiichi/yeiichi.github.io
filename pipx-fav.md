---
layout: default
title: "Nice to have in pipx"
permalink: /pipx-fav
---

# Nice to have in pipx
_2026-04-27_

## pipx install
- python-semantic-release
- sphinx
- ruff
- build
- twine

FYI: [Share your pipx list](https://www.reddit.com/r/Python/comments/yegk2m/share_your_pipx_list/) on Reddit

## uv / uvx alternatives
- uv build                    # build using project environment
- uvx twine check dist/*     # ephemeral execution
- uvx twine upload dist/*

Note: Prefer pipx-installed tools for daily use; use uvx for ad-hoc or CI-style execution.