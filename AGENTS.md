# AGENTS.md — Image Prompt Generator

This repository is dedicated to composing, extracting, revising, and validating photorealistic image-generator prompts from user instructions and reference images.

## Mandatory workflow for image prompt requests

- For requests about an image prompt, image reference analysis, avatar face preservation, changing pose/outfit/background/lighting, JSON prompts, or Leonardo AI conversion, first read `.agents/skills/image-prompt-architect/SKILL.md` and follow its rules.
- When the skill mentions reference templates or tests, read them from `.agents/skills/image-prompt-architect/references/` when helpful.
- Prefer the user's most recent explicit changes. Keep the avatar identity from the avatar reference, and do not import the pose-reference subject's identity.
- Return a valid JSON prompt by default, unless the user requests platform-specific plain text (such as Leonardo AI).
- Preserve all unmentioned properties during minor prompt edits. Ask only for genuinely essential missing information; otherwise use conservative defaults and identify assumptions.
- Camera values are illustrative recommendations, never claimed as verified EXIF.
- Text tokens like `/preserve-face` are descriptive instructions; do not claim they guarantee face preservation or are supported natively by every engine.
- The skill generates prompts, not final images. Don't claim an image has been generated when only a text prompt was produced.

## For Codex Cloud / Web

In some cloud environments, repository skills may not appear in an automatically discovered skills selector. Do not treat this as a missing file. Follow the explicit path above, reading `SKILL.md` directly whenever the task matches.

## Other requests

For repository maintenance requests unrelated to image prompts, treat the skill as context rather than forcing its output format.
