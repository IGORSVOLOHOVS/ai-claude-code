# ai-claude-code

The public home page and privacy policy of AI Claude Code, a personal assistant
that one person, Igors Volohovs, runs on his own computer. On his command it
creates entries in his own Google Tasks and Google Calendar, including Google
Meet links, and reads them back. It is not offered to anyone else. These two
pages exist for its Google OAuth application.

## What is here

- `index.html` is the home page.
- `privacy.html` is the privacy policy: what the application accesses, how the
  data is used, where it is kept, and how access is revoked.

Both are static HTML with their style inline. There is no build step, no script
and no dependency. GitHub Pages serves them from the GitHub copy of this
repository at <https://igorsvolohovs.github.io/ai-claude-code/>. Nothing pushes
from this Gitea copy to GitHub, so a change made here does not reach that
address by itself.

## Working on it

This repository ships no program. `.\recipe.ps1` is still its whole interface:
`.\recipe.ps1 test` runs pytest and ruff over `.claude`, and `.\recipe.ps1 info`
prints the metadata. The other verbs print `nothing to <verb>`. `CLAUDE.md`
holds the rules.

## License

MIT. See [LICENSE](LICENSE).
