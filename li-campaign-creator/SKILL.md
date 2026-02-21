---
name: li-campaign-creator
description: Creates LinkedIn (+ Twitter/X) multi-post campaigns for engineers and technical leaders showcasing developer tools, projects, or AI journeys. Use this when asked to create a LinkedIn campaign, a social media campaign, or a series of posts about developer tools, engineering work, or a personal/professional brand story. Also use when asked to create a campaign plan, a post series, or a content calendar for LinkedIn.
---

# LinkedIn Campaign Creator — for Engineers and Technical Leaders

This skill creates punchy, paste-ready, multi-post LinkedIn (and optional Twitter/X) campaigns. It has been refined through a real campaign built for **Garrard Kitchen** (Fujitsu Distinguished Engineer, Cloud Native Lead) and captures all format rules, structural patterns, and editorial decisions that emerged from that process.

---

## 1. BEFORE STARTING — ASK THESE QUESTIONS

Before writing a single post, ask the user:

1. **Platform**: LinkedIn only, or LinkedIn + Twitter/X thread?
2. **Topic / subject**: What is the campaign about? Tools, projects, a journey, a launch?
3. **Post count**: How many posts? (Default: 1 origin + 1 per subject item + 1 closing = N+2)
4. **Cadence**: Every day, every other day, or custom?
5. **Audience**: Broad technical, specific niche (e.g. Azure engineers), leadership, community?
6. **Tone**: Personal + professional mix, or purely professional?
7. **CTA goal**: Try/download, follow, share, reach out, or a combination?
8. **Visuals**: Will the user create screenshots / graphics? (Needed for code snippets.)
9. **Anything sensitive**: Are there topics the user wants to avoid or handle carefully?

Only proceed once you have enough to write compellingly. If the user provides rich detail upfront, skip questions already answered.

---

## 2. CAMPAIGN STRUCTURE

A campaign follows this arc:

```
Post 1  — Origin story (how it all started, the why behind the work)
Post 2  — Subject item 1 (tool / project / achievement)
Post 3  — Subject item 2
...
Post N  — Subject item N
Post N+1 — Closing rally (reflection, what's next, full CTA)
```

Posts are published every other day (Day 1, Day 3, Day 5 …).

Each post must:
- Open with a hook (single punchy sentence or fragment — no opener like "I'm excited to share")
- Deliver value quickly (the problem, the insight, the capability)
- End with a forward tease to the next post
- Include a CTA appropriate to that post
- Stay under ~1,200 characters for LinkedIn body (longer is fine for special posts but prefer punchy)

---

## 3. LINKEDIN FORMAT RULES — CRITICAL

**LinkedIn does NOT render Markdown.** Every post must be paste-ready plain text.

### What to NEVER use in the LinkedIn section:
- `**bold**` or `*italic*` — renders as literal asterisks
- ` ```code blocks``` ` — renders as literal backticks
- `` `inline code` `` — renders as literal backtick
- `## headers` — renders as literal `##`
- `- bullet points` — use `→` instead

### What TO use:
- `→` for bullet points / feature lists
- `─────` (5+ em dashes) for section dividers
- Emoji for visual punch (⚡ 🔐 🔍 ☁️ 🧵 etc.) — use sparingly, purposefully
- Plain numbered lists where order matters
- ALL CAPS for emphasis (use very sparingly — one word max)
- Line breaks for pacing

### Code snippets in LinkedIn posts:
Multi-line code blocks CANNOT be pasted into LinkedIn. Handle them as follows:
- Single-line commands: paste inline as plain text (no backticks)
- Multi-line blocks (docker run, docker-compose, .env files): flag as `📸 [attach screenshot]` with note to use carbon.now.sh or similar
- Always add the relevant doc link immediately below any code snippet

---

## 4. TWITTER/X THREAD FORMAT

Twitter/X threads are optional but add reach. Structure:

```
[1/N] Hook tweet — grabs attention, teases the thread
[2/N] The problem / context
[3/N] The solution / what you built
[4/N] Key capabilities (→ bullets, max 4 per tweet)
[5/N] Technical detail or security/trust angle
[6/N] Teaser to next post in the campaign
[N/N] Full CTA + hashtags + doc link
```

Keep each tweet under 280 characters. The hashtag block only appears on the final tweet.

---

## 5. FILE STRUCTURE

Save each post to its own file in the working directory:

```
post-01-origin.md
post-02-[tool-name].md
post-03-[tool-name].md
...
post-N-closing.md
visual-asset-brief.md
```

Each file contains:
1. A header comment block identifying the post number, platform, and publish day
2. The LinkedIn section (paste-ready plain text, clearly labelled)
3. The Twitter/X thread section (if applicable, clearly labelled)
4. A visual asset brief section (screenshot guidance, badge suggestions, image dimensions)

---

## 6. INSTALL SNIPPETS — FOR TOOL POSTS

When a post covers a developer tool, include the install/run command in the LinkedIn post body. Rules:

- **Single-line commands**: paste inline as plain text (no backticks)
- **Multi-line commands**: flag as screenshot attachment
- Always include a link to the exact install/getting-started doc page (not just the homepage)
- If install varies by OS (macOS / Windows / Linux): list all variants or link to the page that covers them all

Example format in a LinkedIn post:

```
⚡ Try it now:

dotnet tool install -g EntraAuthCli

Full install guide (all platforms):
👉 https://docs.example.com/getting-started/installation/
```

---

## 7. PRIVACY & DATA GUARANTEE — FOR FREE TOOLS

If the campaign covers free developer tools built by the author, include this guarantee in each tool post (LinkedIn body, before the day teaser):

```
A note on every tool I've shared: zero monitoring, zero data harvesting.
Personal guarantee. I built these to help — not to benefit from giving back.
```

On the first tool post (or the closing post), expand this into a full paragraph:

```
One thing I want to be clear on — for every tool I've built and shared:

Zero monitoring. Zero data harvesting. Not now. Not ever.

These tools do not phone home. They do not collect usage data.
They do not track you in any way. You have my personal guarantee on that.
I built these to help, not to benefit from giving back to the community.
```

---

## 8. AUTHOR CONTEXT — GARRARD KITCHEN

When creating campaigns for Garrard Kitchen, use this context:

**Role**: Fujitsu Distinguished Engineer, Cloud Native Lead  
**Day job covers**: architecture, leadership, Azure, DevOps, infrastructure, .NET development, security, testing, code reviews, continuous delivery, and supporting other teams  
**AI tools**: Claude Code CLI (paid/client work), GitHub Copilot CLI (personal projects). Has been exploring AI assistants on his own time and licence for 2+ years.  
**Website**: garrardkitchen.com  
**Campaign title in use**: "An Engineer's AI Journey: Exploring Models, Building Tools, Giving Back"

**Tone rules for Garrard's posts**:
- Authentic and vulnerable — he will admit uncertainty, friction, and honest feelings
- Never boastful or self-promotional in a corporate way
- Specific over vague — names the exact tool, the exact problem, the exact fix
- "Engineer's voice" — precise technical language but accessible writing
- Always shows the human behind the work

**Recurring themes**:
- Filling gaps (building what doesn't exist yet)
- Giving back to the community with no strings attached
- The honest tension between being a developer and feeling more like a product manager
- AI-assisted code and the "icky" feeling of presenting work you didn't write 100% yourself
- The journey from solo experimenter to leading team-wide AI adoption
- Security-first by default (zero keys, zero exposure, supply chain awareness)

---

## 9. SUPPLY CHAIN / SECURITY FRAMING

For tools related to authentication, AI agent tooling, or file sharing, use technically accurate security framing:

| Vague phrase | Accurate alternative |
|---|---|
| "supply chain visibility" | "auditing the full tool surface area your AI agents can invoke" |
| "keeps secrets safe" | "platform-native encryption (Windows DPAPI / macOS Keychain)" |
| "secure file transfer" | "end-to-end AES-256-GCM encrypted, SHA-512 integrity verified" |
| "no keys needed" | "eliminates long-lived credentials — all flows use short-lived tokens" |

---

## 10. CLOSING POST STRUCTURE

The closing rally post (final post in the campaign) must include:

1. **Recap** — one-liner per tool/subject, the full set in order
2. **Reflection** — honest look at what the work meant, how it felt, what was hard
3. **AI/assistance honesty section** — if AI was used to build or write: acknowledge it directly, without apology, with nuance about where the human judgement and ideas came from
4. **Role grounding** — what the author's day job looks like (keeps it real)
5. **What's next** — any transition, next chapter, or forward momentum
6. **Full CTA** — follow, try, share, reach out

The closing post can be longer than other posts — it earns the length.

---

## 11. VISUAL ASSET BRIEF

Always produce a `visual-asset-brief.md` file alongside the posts. It should specify per post:

- Image type (screenshot, diagram, graphic, terminal output)
- Recommended content (what to show, what angle)
- Dimensions (1200x800 or 16:9 for LinkedIn; 1:1 or 16:9 for Twitter)
- Badge / overlay text suggestion
- Caption suggestion
- Tool recommendations: carbon.now.sh for code screenshots, CleanShot X for macOS, Figma for graphics

---

## 12. HASHTAG STRATEGY

- LinkedIn: 5–10 hashtags, placed at the bottom of the post body. Mix broad reach (#DotNet, #Azure, #DevOps) with specific niche (#MCP, #ModelContextProtocol, #EntraID) and campaign-level (#DeveloperTools, #BuildInPublic, #AIJourney).
- Twitter/X: 3–5 hashtags, final tweet only.
- Do NOT use `#OpenSource` for tools that are not open source.

---

## 13. COLLEAGUE & CLIENT SENSITIVITY

Never frame colleagues, clients, or other engineers as the source of a problem. Always attribute friction to the **tooling, the situation, or the industry pattern** — not the people.

| Avoid | Use instead |
|---|---|
| "colleagues were struggling with" | "friction points that came up again and again in project work" |
| "engineers were losing hours to this" | "this was a pattern every sprint — time that should have gone into building" |
| "too many engineers are handing out secrets" | "in practice, secrets end up in places they shouldn't — it's friction, not carelessness" |
| "I watched engineers lose hours" | "this was a recurring pattern on the project" |
| "customers were tripping over" | "a gap that came up repeatedly in client work" |
| "nobody was moving fast enough" | "the pace the project needed wasn't there yet" |

The rule: the problem is always the gap, the tooling, or the situation. The people are capable — they just didn't have the right tool yet.

---

## 14. WRITING QUALITY CHECKLIST

Before outputting any post, verify:

- [ ] No `**`, `*`, ` ``` `, `` ` ``, or `##` in the LinkedIn section
- [ ] All bullets use `→`
- [ ] Multi-line code blocks flagged as screenshot attachments
- [ ] Doc links present for any install/run commands
- [ ] Privacy guarantee included (for free tool campaigns)
- [ ] Forward tease to next post present
- [ ] CTA present
- [ ] Hashtags at bottom (LinkedIn) or final tweet only (Twitter)
- [ ] Post opens with a hook, not "I'm excited to share" or "Today I want to talk about"
