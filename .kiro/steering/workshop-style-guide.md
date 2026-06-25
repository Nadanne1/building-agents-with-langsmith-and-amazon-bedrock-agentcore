---
inclusion: auto
---

# AWS Workshop Content Style Guide

This steering document defines the style, structure, and conventions for all workshop content in this repository. It is modeled after the [AgentCore Deep Dive workshop](https://catalog.workshops.aws/agentcore-deep-dive/en-US) and should be followed for all new and updated content.

## File Structure

### Numbering Convention
- Use **10-increment numbering** for major sections: `00-prerequisites`, `10-fundamentals`, `20-lab-name`
- This leaves room to insert new sections without renumbering

### Frontmatter
Every `index.en.md` must have:
```yaml
---
title: "Page Title"
weight: <number>
---
```

## Writing Style

### Tone
- Professional but accessible. Write as a knowledgeable colleague, not a textbook.
- Use "you" to address the reader directly.
- Explain the **why** before the **how**.
- Be honest about workarounds and limitations.

### Page Structure
1. **Title** — H1 heading
2. **Overview** — what this lab covers and why it matters
3. **Body** — steps with code blocks
4. **Checkpoint** — confirm what the reader accomplished

### Code Blocks
- Always specify the language: ```bash, ```python, ```yaml
- Don't mix commands and output in the same block

### Notice Blocks
```
{{% notice info %}} / {{% notice tip %}} / {{% notice warning %}} / {{% notice note %}}
```

### Checkpoints
Every lab ends with:
```
{{% notice tip %}}
**Checkpoint:** You should have [specific observable outcome].
{{% /notice %}}
```
