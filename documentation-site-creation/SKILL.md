---
name: documentation-site-creation
description: Create comprehensive documentation and training guide sites using Hugo and modern documentation themes
applyTo: '**'
tags:
  - documentation
  - hugo
  - static-site
  - training-guides
  - github-pages
---

# Skill: Creating Comprehensive Documentation/Training Guide Sites

## Overview

This skill captures the methodology, tools, and best practices for creating professional documentation and training guide websites for software applications using Hugo and modern documentation themes.

---

## When to Use This Skill

- Creating user documentation for software applications
- Building training guides for complex tools
- Setting up developer documentation sites
- Migrating from README-only docs to full documentation sites
- Establishing a learning portal for enterprise software

---

## Technology Stack

### Core Tools

**Hugo** (Static Site Generator)
- Version: 0.100.0+ (extended version required)
- Purpose: Fast, flexible static site generation
- Why: Simple to use, great themes, excellent build performance

**Lotus Docs Theme** (or similar documentation theme)
- Repository: https://github.com/colinwilson/lotusdocs
- Purpose: Professional documentation layout
- Alternatives: Docsy, Book theme, Learn theme

**GitHub Pages** (Hosting)
- Purpose: Free hosting with custom domains
- Why: Integrated with GitHub, automatic deployments, HTTPS support

**GitHub Actions** (CI/CD)
- Purpose: Automated building and deployment
- Why: Integrated with repository, easy configuration

### Supporting Tools

- **Go**: Required for Hugo modules
- **Git**: Version control and deployment
- **Markdown**: Content format
- **YAML**: Configuration format

---

## Methodology

### Phase 1: Planning & Discovery

#### 1.1 Gather Information

**Extract from existing sources:**
- README.md files
- CHANGELOG.md files
- Code comments and documentation
- Existing user guides
- GitHub issues/discussions

**Questions to ask the stakeholder:**
1. What's the target audience? (beginners, developers, mixed)
2. What are the top 5 most important features?
3. Do you have a custom domain?
4. What's the GitHub repository structure?
5. Any specific branding requirements? (colors, logo)
6. What level of detail is needed? (brief, moderate, detailed)
7. Are there existing screenshots or should they be created?

**Deliverable:** Requirements document with clear answers

---

#### 1.2 Identify Features

**Map the application structure:**
- Main features (shown as tabs, pages, or menu items)
- Sub-features within each main feature
- Common workflows and use cases
- Advanced or admin-only features

**Categorize features:**
- **Core features**: Must document in detail
- **Supporting features**: Moderate documentation
- **Advanced features**: Brief documentation with links

**Deliverable:** Feature hierarchy and priority list

---

### Phase 2: Site Setup

#### 2.1 Initialize Hugo Site

```bash
# Navigate to docs directory
cd docs

# Create Hugo site
hugo new site learn --format yaml

# Initialize Hugo modules (recommended approach)
cd learn
hugo mod init github.com/{owner}/{repo}-docs
```

**Note**: Modern Hugo themes like Lotus Docs use **Hugo modules** rather than git submodules. This approach is cleaner and avoids git submodule complications in CI/CD pipelines.

#### 2.2 Configure Hugo

**Create/update `hugo.yaml`:**

```yaml
baseURL: "https://docs.example.com/"
title: "Application Name Documentation"
languageCode: "en-us"
enableEmoji: true
enableGitInfo: false

# Hugo Modules (if required by theme)
module:
  hugoVersion:
    extended: true
    min: "0.100.0"
  imports:
    - path: github.com/colinwilson/lotusdocs
    - path: github.com/gohugoio/hugo-mod-bootstrap-scss/v5

# Site Parameters
params:
  description: "Brief site description for SEO"
  author: "Your Name/Company"
  
  # Branding (customize to match app)
  primary: "#0066cc"      # Primary brand color
  secondary: "#003d7a"    # Secondary brand color
  
  # Social links
  social:
    github: "https://github.com/{owner}/{repo}"
  
  # Docs configuration
  docs:
    title: "Documentation"
    pathName: "docs"
    themeColor: "blue"
  
  # Table of Contents
  toc:
    enable: true
    startLevel: 2
    endLevel: 4
  
  # Footer
  footer:
    copyright: "© :YEAR: Company Name"
    version: true

# Menu Configuration
menu:
  primary:
    - name: Home
      url: /
      weight: 10
    - name: Documentation
      url: /docs/
      weight: 20
    - name: GitHub
      url: https://github.com/{owner}/{repo}
      weight: 30

# Markup Configuration
markup:
  goldmark:
    renderer:
      unsafe: true
  highlight:
    style: monokai
    lineNos: true
```

#### 2.3 Create Directory Structure

```bash
cd content
mkdir -p docs/getting-started
mkdir -p docs/features
mkdir -p docs/advanced
mkdir -p docs/reference
```

**Standard structure:**
```
content/
├── _index.md                    # Home page
└── docs/
    ├── _index.md               # Docs index
    ├── getting-started/
    │   ├── installation.md
    │   ├── quickstart.md
    │   └── concepts.md
    ├── features/
    │   ├── feature1.md
    │   ├── feature2.md
    │   └── feature3.md
    ├── advanced/
    │   ├── configuration.md
    │   └── troubleshooting.md
    └── reference/
        ├── api.md
        └── faq.md
```

---

### Phase 3: Content Creation

#### 3.1 Restructuring Large Documentation Files

When documentation grows, large single files with many sections become difficult to navigate and maintain. The solution is to split content into multiple focused files organized in subdirectories.

##### 3.1.1 When to Restructure

**Signs that documentation needs splitting:**
- Single file exceeds 500-1000 lines
- Multiple distinct topics in one file
- Table of contents becomes unwieldy
- Users report difficulty finding information
- Many internal anchor links within one file
- File takes significant time to load/render

**Before restructuring:**
```
content/docs/
├── recipes.md              # 1500 lines, 12 recipes
├── certificates.md         # 800 lines, 8 topics
├── oauth-flows.md          # 1200 lines, 5 flows
├── platform-guides.md      # 1000 lines, 3 platforms
├── reference.md            # 2000 lines, 10 commands
└── troubleshooting.md      # 900 lines, 8 topics
```

**After restructuring:**
```
content/docs/
├── recipes/
│   ├── _index.md          # Landing page with panels
│   ├── microsoft-graph.md
│   ├── azure-management.md
│   ├── cicd-integration.md
│   └── ...
├── certificates/
│   ├── _index.md
│   ├── overview.md
│   ├── creating.md
│   └── ...
├── oauth-flows/
│   ├── _index.md
│   ├── client-credentials.md
│   ├── authorization-code.md
│   └── ...
└── ...
```

##### 3.1.2 Creating _index.md Landing Pages with Panels

The `_index.md` file serves as a landing page that provides overview and navigation to individual topic files.

**Standard _index.md structure:**

```markdown
---
title: "Section Name"
description: "Brief section overview"
weight: 10
---

# Section Name

Brief introduction explaining what this section covers and why it's important.

## Overview

2-3 sentences about the content in this section and when users should refer to it.

---

## Topics

{{</* cards */>}}
  {{</* card link="topic-one" title="Topic One" icon="icon-name" */>}}
  Brief description of what this topic covers. 1-2 sentences max.
  {{</* /card */>}}
  
  {{</* card link="topic-two" title="Topic Two" icon="icon-name" */>}}
  Brief description of what this topic covers. 1-2 sentences max.
  {{</* /card */>}}
  
  {{</* card link="topic-three" title="Topic Three" icon="icon-name" */>}}
  Brief description of what this topic covers. 1-2 sentences max.
  {{</* /card */>}}
{{</* /cards */>}}

---

## Quick Reference

Optional comparison table or quick reference for the topics:

| Topic | Use Case | Difficulty |
|-------|----------|------------|
| Topic One | Use case description | Beginner |
| Topic Two | Use case description | Intermediate |
| Topic Three | Use case description | Advanced |
```

**Important Notes:**
- Theme-specific shortcodes: Check your theme's documentation for panel/card shortcodes
- Lotus Docs uses `{{</* cards */>}}` and `{{</* card */>}}`
- Some themes use different shortcode names
- **Fallback**: If shortcodes unavailable, use standard markdown lists with links
- Icons: Optional but improve visual appeal

**Example with standard markdown (no shortcodes):**

```markdown
---
title: "OAuth Flows"
description: "Authentication flow implementations"
weight: 30
---

# OAuth Flows

Choose the right OAuth 2.0 flow for your authentication scenario.

## Available Flows

### 🔐 [Client Credentials Flow](client-credentials/)
Service-to-service authentication without user interaction. Ideal for backend services, APIs, and daemon applications.

### 🌐 [Authorization Code Flow](authorization-code/)
Web application authentication with user login. Best for web apps that need access to user data with secure token handling.

### 📱 [Device Code Flow](device-code/)
Authentication for devices with limited input. Perfect for IoT devices, smart TVs, and command-line tools.

### 🖥️ [Interactive Browser Flow](interactive-browser/)
CLI tools with browser-based authentication. Provides the best user experience for desktop applications.

---

## Comparison

| Flow | Use Case | User Interaction | Token Type |
|------|----------|------------------|------------|
| Client Credentials | Backend services | None | Access token only |
| Authorization Code | Web applications | Browser login | Access + Refresh |
| Device Code | Limited input devices | Remote browser | Access + Refresh |
| Interactive Browser | CLI tools | Local browser | Access + Refresh |
```

##### 3.1.3 Splitting Process

**Step-by-step approach:**

1. **Analyze the original file structure**
   ```bash
   # Identify major sections and subsections
   grep "^##" recipes.md
   ```

2. **Create subdirectory and _index.md**
   ```bash
   mkdir -p content/docs/recipes
   touch content/docs/recipes/_index.md
   ```

3. **Create individual topic files**
   - Extract each major section into its own file
   - Use descriptive kebab-case filenames: `microsoft-graph.md`, `cicd-integration.md`
   - Keep original heading structure within each file

4. **Build _index.md landing page**
   - Write overview introduction
   - Create panel/card navigation to each topic
   - Add comparison table if helpful
   - Include quick reference section

5. **Set appropriate weights**
   ```yaml
   # _index.md
   weight: 30  # Section position in main menu
   
   # microsoft-graph.md
   weight: 10  # First topic in section
   
   # azure-management.md
   weight: 20  # Second topic in section
   ```

6. **Update internal links**
   ```markdown
   # Before (within single file)
   See [Certificates](#certificate-configuration)
   
   # After (across multiple files)
   See [Certificates](../certificates/configuration/)
   ```

7. **Delete original single file**
   ```bash
   rm content/docs/recipes.md
   ```

8. **Verify Hugo build**
   ```bash
   hugo --gc
   # Check for broken links, missing pages
   ```

##### 3.1.4 Topic File Best Practices

**Each topic file should:**
- Have clear, descriptive title
- Start with brief overview (2-3 sentences)
- Use consistent heading structure (##, ###, ####)
- Include practical examples
- Have proper YAML front matter with weight
- Be self-contained (minimal cross-references)

**Standard topic file template:**

```markdown
---
title: "Specific Topic Name"
description: "What this topic covers"
weight: 10
---

# Specific Topic Name

Brief 2-3 sentence overview of this topic.

## Prerequisites

- Requirement 1
- Requirement 2

## Overview

Detailed explanation of what this topic covers and why it's useful.

## Implementation

### Step 1: First Action

Detailed instructions...

```bash
# Code example
command --flag value
```

### Step 2: Second Action

More instructions...

## Examples

### Basic Example

```bash
# Simple use case
command example
```

### Advanced Example

```bash
# Complex use case
advanced command example
```

## Common Issues

### Issue Name

**Problem:** Description of the problem

**Solution:** How to fix it

## See Also

- [Related Topic 1](../related-topic-1/)
- [Related Topic 2](../related-topic-2/)
```

##### 3.1.5 Command Name Corrections

After restructuring, use global search-replace for consistency:

```bash
# Find all occurrences
grep -r "old-command" content/docs/section-name/

# Replace across all new files (macOS/BSD sed)
find content/docs/section-name -type f -name "*.md" ! -name "_index.md" \
  -exec sed -i '' 's/old-command/new-command/g' {} +

# Update _index.md files separately
sed -i '' 's/old-command/new-command/g' \
  content/docs/*/\_index.md

# Verify changes
grep -r "new-command" content/docs/section-name/ | wc -l
```

##### 3.1.6 Testing Restructured Documentation

**Checklist after restructuring:**
- [ ] Hugo builds without errors (`hugo --gc`)
- [ ] All _index.md pages render with navigation panels
- [ ] Individual topic pages load correctly
- [ ] Navigation menu shows section properly
- [ ] Internal links work (no 404s)
- [ ] Search finds content in new locations
- [ ] Mobile navigation works
- [ ] Breadcrumbs display correctly
- [ ] Page count increased appropriately (hugo output)

**Validation commands:**

```bash
# Build and check page count
hugo --gc 2>&1 | grep "Pages"
# Should show increased page count

# Check for broken links
hugo --gc 2>&1 | grep -i "error\|warn"

# Test local server
hugo server -D --bind 127.0.0.1 --port 1313
# Navigate to restructured sections
```

##### 3.1.7 Benefits of This Structure

**Improved user experience:**
- Faster page loads (smaller files)
- Clearer topic separation
- Better navigation with landing pages
- Easier to find specific information
- Visual panels guide users

**Better maintenance:**
- Easier to update individual topics
- Clear organization for contributors
- Simpler to add new topics
- Reduced merge conflicts
- Better version control diffs

**Enhanced SEO:**
- Each topic has unique URL
- More specific page titles
- Better internal linking structure
- Improved crawlability

---

#### 3.2 Content Structure Template

**Every documentation page should have:**

```markdown
---
title: "Feature Name"
description: "Brief description for SEO and previews"
weight: 1  # Lower numbers appear first
---

## Overview

Brief 2-3 sentence overview of what this feature does.

---

## Key Capabilities

- ✅ Capability 1
- 🔍 Capability 2
- ⭐ Capability 3
- 📝 Capability 4

---

## Getting Started

### Step 1: First Action

1. Navigate to X
2. Click Y
3. Fill in Z

**📸 Screenshot needed:** `descriptive-name.png`
*Description: What should be shown in this screenshot*

---

### Step 2: Second Action

[Content here]

---

## Advanced Features

### Feature A

[Content here]

---

## Common Workflows

### Workflow 1: Name

1. Step one
2. Step two
3. Step three

---

### Workflow 2: Name

[Content here]

---

## Troubleshooting

### Problem 1

**Problem**: Description

**Solutions:**
1. First solution
2. Second solution
3. Third solution

---

## Related Pages

- [Related Feature 1](/docs/features/feature1/)
- [Related Feature 2](/docs/features/feature2/)
```

#### 3.2 Content Writing Guidelines

**Voice & Tone:**
- Use second person ("you") for instructions
- Be clear, concise, and direct
- Avoid jargon unless necessary
- Define technical terms on first use

**Structure:**
- **Short paragraphs**: 2-3 sentences max
- **Bullet points**: For lists and options
- **Numbered lists**: For sequential steps
- **Headers**: Clear hierarchy (H2 for sections, H3 for sub-sections)
- **Emphasis**: Bold for UI elements, italic for emphasis

**Visual Elements:**
- Use emojis for quick visual scanning
- Include screenshots at every key step
- Add code blocks with syntax highlighting
- Use callout boxes for important notes

**Example patterns:**

```markdown
> **info:** This is an informational callout

> **warning:** This is a warning callout

> **danger:** This is a critical warning
```

#### 3.3 Screenshot Strategy

**Planning:**
1. List all features requiring visual documentation
2. Create descriptive filename for each (use kebab-case)
3. Document what each screenshot should show
4. Organize capture workflow by feature

**Naming convention:**
```
{feature}-{description}.png

Examples:
✅ tools-list-sidebar.png
✅ connection-form-basic.png
✅ chat-model-selection.png

❌ screenshot1.png
❌ Tools_List.png
```

**Capture guidelines:**
- Consistent browser and window size
- Same zoom level (100%)
- Clean UI (close unnecessary elements)
- Use realistic but safe data
- Light OR dark mode (be consistent)
- Compress large images (< 500KB)

**Placeholders in documentation:**
```markdown
**📸 Screenshot needed:** `filename.png`
*Description: Clear description of what to capture*
```

---

### Phase 3.5: Customizing the Home Page (Optional)

#### 3.5.1 When to Replace the Default Theme Homepage

**Consider creating a custom homepage when:**
- The default Lotus Docs homepage doesn't match your branding
- You want a marketing-focused landing page rather than documentation-first
- You need specific sections (hero, features grid, quick start, etc.)
- You want full control over layout and styling
- Your application needs a distinctive first impression

**Keep the default homepage when:**
- The theme's layout suits your needs
- You prefer configuration over custom code
- Time/resources are limited
- You want automatic theme updates to affect homepage

#### 3.5.2 Hugo Layout Override System

Hugo uses a template hierarchy where **your local layouts override theme layouts**:

```
layouts/
  ├── index.html          # YOUR custom homepage (overrides theme)
  └── _default/
      └── baseof.html     # YOUR custom base template (if needed)

themes/lotusdocs/layouts/  # Theme's defaults (ignored if you have local)
  ├── index.html          # Theme's homepage
  └── _default/
```

**Key principle**: Create `layouts/index.html` to completely replace the theme's homepage.

#### 3.5.3 Custom Homepage Template Structure

Create `layouts/index.html` with this proven structure:

```html
{{ define "main" }}
<div class="app-home">
  <!-- Hero Section -->
  <section class="hero">
    <div class="hero-content">
      <h1>{{ .Title }}</h1>
      <p class="tagline">Brief compelling description of your application</p>
      <div class="cta-buttons">
        <a href="/docs/getting-started/" class="btn btn-primary">Get Started</a>
        <a href="/docs/features/" class="btn btn-secondary">Explore Features</a>
        <a href="https://github.com/owner/repo" class="btn btn-outline" target="_blank">
          <!-- GitHub SVG icon -->
          <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
            <path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/>
          </svg>
          GitHub
        </a>
      </div>
    </div>
  </section>

  <!-- Key Features Grid -->
  <section class="key-features">
    <h2>Key Features</h2>
    <div class="features-grid">
      <!-- 6 feature cards -->
      <div class="feature-card">
        <div class="feature-icon">🎯</div>
        <h3>Feature Name</h3>
        <p>Brief description of the feature and its benefits.</p>
        <a href="/docs/features/feature1/">Learn more →</a>
      </div>
      <!-- Repeat for 5 more features -->
    </div>
  </section>

  <!-- Why Section -->
  <section class="why-section">
    <h2>Why [App Name]?</h2>
    <div class="benefits-grid">
      <div class="benefit">
        <div class="benefit-icon">🚀</div>
        <h3>Benefit Title</h3>
        <p>Explanation of this benefit.</p>
      </div>
      <!-- 2 more benefits -->
    </div>
  </section>

  <!-- Quick Start -->
  <section class="quick-start">
    <h2>Quick Start</h2>
    <div class="steps">
      <div class="step">
        <div class="step-number">1</div>
        <h3>Step Title</h3>
        <p>Step description</p>
        <pre><code>command example</code></pre>
      </div>
      <!-- 3 more steps -->
    </div>
  </section>

  <!-- Top 5 Features (detailed) -->
  <section class="top-features">
    <h2>Top 5 Features</h2>
    <div class="top-features-list">
      <div class="top-feature">
        <span class="rank">1</span>
        <div class="content">
          <h3>Feature Name</h3>
          <p>Detailed description with benefits and use cases.</p>
        </div>
      </div>
      <!-- 4 more features -->
    </div>
  </section>

  <!-- Call to Action -->
  <section class="cta-section">
    <h2>Ready to [Action]?</h2>
    <p>Compelling message to get started</p>
    <div class="cta-buttons">
      <a href="/docs/getting-started/" class="btn btn-primary btn-lg">Getting Started</a>
      <a href="/docs/features/" class="btn btn-secondary btn-lg">Browse Features</a>
    </div>
  </section>
</div>

<style>
/* Include all CSS styles inline */
.app-home {
  width: 100%;
}

/* Hero Section */
.hero {
  background: linear-gradient(135deg, #0066cc 0%, #003d7a 100%);
  color: white;
  padding: 80px 20px;
  text-align: center;
  border-radius: 8px;
  margin-bottom: 60px;
}

.hero-content h1 {
  font-size: 3.5rem;
  margin-bottom: 20px;
  font-weight: 700;
}

.hero-content .tagline {
  font-size: 1.5rem;
  margin-bottom: 40px;
  opacity: 0.95;
}

.cta-buttons {
  display: flex;
  gap: 15px;
  justify-content: center;
  flex-wrap: wrap;
}

.btn {
  padding: 12px 30px;
  border-radius: 6px;
  text-decoration: none;
  font-weight: 600;
  transition: all 0.3s ease;
  display: inline-flex;
  align-items: center;
  gap: 8px;
}

.btn-primary {
  background: white;
  color: #0066cc;
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.2);
}

.btn-secondary {
  background: rgba(255,255,255,0.2);
  color: white;
  border: 2px solid white;
}

.btn-secondary:hover {
  background: rgba(255,255,255,0.3);
}

.btn-outline {
  background: transparent;
  color: white;
  border: 2px solid white;
}

.btn-outline:hover {
  background: white;
  color: #0066cc;
}

/* Key Features Grid */
.key-features {
  margin-bottom: 80px;
}

.key-features h2 {
  text-align: center;
  font-size: 2.5rem;
  margin-bottom: 50px;
}

.features-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 30px;
}

.feature-card {
  background: white;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 30px;
  transition: all 0.3s ease;
}

.feature-card:hover {
  box-shadow: 0 8px 24px rgba(0,0,0,0.1);
  transform: translateY(-4px);
}

.feature-icon {
  font-size: 3rem;
  margin-bottom: 15px;
}

.feature-card h3 {
  margin-bottom: 15px;
  color: #0066cc;
}

.feature-card p {
  margin-bottom: 15px;
  color: #666;
}

.feature-card a {
  color: #0066cc;
  text-decoration: none;
  font-weight: 600;
}

.feature-card a:hover {
  text-decoration: underline;
}

/* Why Section */
.why-section {
  background: #f8f9fa;
  padding: 60px 20px;
  border-radius: 8px;
  margin-bottom: 80px;
}

.why-section h2 {
  text-align: center;
  font-size: 2.5rem;
  margin-bottom: 50px;
}

.benefits-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 40px;
  max-width: 1200px;
  margin: 0 auto;
}

.benefit {
  text-align: center;
}

.benefit-icon {
  font-size: 3rem;
  margin-bottom: 20px;
}

.benefit h3 {
  margin-bottom: 15px;
  color: #0066cc;
}

.benefit p {
  color: #666;
}

/* Quick Start */
.quick-start {
  margin-bottom: 80px;
}

.quick-start h2 {
  text-align: center;
  font-size: 2.5rem;
  margin-bottom: 50px;
}

.steps {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 30px;
}

.step {
  background: white;
  border: 2px solid #0066cc;
  border-radius: 8px;
  padding: 30px;
  position: relative;
}

.step-number {
  position: absolute;
  top: -20px;
  left: 20px;
  background: #0066cc;
  color: white;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 1.2rem;
}

.step h3 {
  margin-top: 10px;
  margin-bottom: 15px;
  color: #0066cc;
}

.step pre {
  background: #f5f5f5;
  padding: 15px;
  border-radius: 4px;
  overflow-x: auto;
}

.step code {
  font-family: 'Courier New', monospace;
  font-size: 0.9rem;
}

/* Top Features */
.top-features {
  margin-bottom: 80px;
}

.top-features h2 {
  text-align: center;
  font-size: 2.5rem;
  margin-bottom: 50px;
}

.top-features-list {
  display: flex;
  flex-direction: column;
  gap: 20px;
  max-width: 800px;
  margin: 0 auto;
}

.top-feature {
  display: flex;
  gap: 20px;
  background: white;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 25px;
  transition: all 0.3s ease;
}

.top-feature:hover {
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  transform: translateX(5px);
}

.top-feature .rank {
  background: #0066cc;
  color: white;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 1.5rem;
  flex-shrink: 0;
}

.top-feature .content h3 {
  margin-bottom: 10px;
  color: #0066cc;
}

.top-feature .content p {
  color: #666;
  margin: 0;
}

/* CTA Section */
.cta-section {
  background: linear-gradient(135deg, #0066cc 0%, #003d7a 100%);
  color: white;
  padding: 60px 20px;
  text-align: center;
  border-radius: 8px;
  margin-bottom: 40px;
}

.cta-section h2 {
  font-size: 2.5rem;
  margin-bottom: 15px;
}

.cta-section p {
  font-size: 1.2rem;
  margin-bottom: 30px;
  opacity: 0.95;
}

.btn-lg {
  padding: 15px 40px;
  font-size: 1.1rem;
}

/* Responsive */
@media (max-width: 768px) {
  .hero-content h1 {
    font-size: 2.5rem;
  }
  
  .hero-content .tagline {
    font-size: 1.2rem;
  }
  
  .features-grid,
  .benefits-grid,
  .steps {
    grid-template-columns: 1fr;
  }
  
  .cta-buttons {
    flex-direction: column;
  }
  
  .btn {
    width: 100%;
    justify-content: center;
  }
}
</style>
{{ end }}
```

#### 3.5.4 Section-by-Section Customization Guide

**Hero Section:**
- **Purpose**: Grab attention, show value proposition
- **Customize**: Title, tagline, button URLs
- **Colors**: Adjust gradient to match brand colors
- **Tip**: Keep tagline under 15 words

**Key Features Grid (6 cards):**
- **Purpose**: Quick overview of main capabilities
- **Customize**: Icons (emojis or Font Awesome), titles, descriptions, links
- **Layout**: Auto-responsive grid (3 columns → 2 → 1)
- **Tip**: Use action verbs in feature titles

**Why Section (3 benefits):**
- **Purpose**: Differentiate from competitors
- **Customize**: Benefits that matter to your users
- **Icons**: Use emojis for visual interest
- **Tip**: Focus on outcomes, not features

**Quick Start (4 steps):**
- **Purpose**: Get users to success quickly
- **Customize**: Step titles, descriptions, code examples
- **Numbers**: Automatically styled circular badges
- **Tip**: Each step should take <5 minutes

**Top 5 Features (detailed):**
- **Purpose**: Deep dive into standout capabilities
- **Customize**: Feature order, titles, descriptions
- **Ranking**: Numbered circles for visual hierarchy
- **Tip**: Include specific use cases

**Call to Action:**
- **Purpose**: Convert visitors to users
- **Customize**: Heading, message, button text/URLs
- **Style**: Matches hero gradient for consistency
- **Tip**: Use action-oriented language

#### 3.5.5 Styling Best Practices

**Colors:**
```css
/* Replace these with your brand colors */
.hero {
  background: linear-gradient(135deg, #YOUR_PRIMARY 0%, #YOUR_SECONDARY 100%);
}

.feature-card h3,
.benefit h3,
.step-number,
.top-feature .rank {
  color: #YOUR_PRIMARY;
  background: #YOUR_PRIMARY; /* for circles */
}
```

**Hover Effects:**
```css
/* Cards lift on hover for interactivity */
.feature-card:hover {
  box-shadow: 0 8px 24px rgba(0,0,0,0.1);
  transform: translateY(-4px);
}
```

**Responsive Breakpoints:**
```css
@media (max-width: 768px) {
  /* Stack grids vertically */
  .features-grid,
  .benefits-grid,
  .steps {
    grid-template-columns: 1fr;
  }
}
```

#### 3.5.6 Content vs. Layout Decision

**Two approaches for homepage content:**

**Approach A: Hardcoded HTML (Recommended)**
- ✅ Full control over layout
- ✅ Can use complex HTML structures
- ✅ Better for marketing-focused pages
- ❌ Changes require editing HTML file
- **Use when**: Homepage is design-focused

**Approach B: Markdown with Custom Layout**
- ✅ Non-technical editors can update
- ✅ Content separate from presentation
- ❌ Limited layout flexibility
- ❌ More complex template logic
- **Use when**: Homepage content changes frequently

**Our implementation uses Approach A** because:
1. Homepage is stable (doesn't change often)
2. Needs precise layout control
3. Includes complex nested structures (grids, cards, steps)
4. Marketing-focused rather than documentation-first

#### 3.5.7 Testing Your Custom Homepage

**Checklist:**
```bash
# 1. Test local build
cd docs/learn
hugo server -D
# Open http://localhost:1313/

# 2. Check all sections render
- [ ] Hero section displays with buttons
- [ ] Features grid shows all 6 cards
- [ ] Why section has 3 benefits
- [ ] Quick Start shows 4 numbered steps
- [ ] Top 5 Features are ranked 1-5
- [ ] CTA section at bottom

# 3. Test responsive design
- [ ] Desktop (1920px+)
- [ ] Tablet (768px-1024px)
- [ ] Mobile (320px-767px)

# 4. Test interactions
- [ ] All buttons link correctly
- [ ] Cards have hover effects
- [ ] External links open in new tab
- [ ] Smooth transitions work

# 5. Test in production
hugo --gc --minify
# Check public/index.html
```

#### 3.5.8 Common Customization Patterns

**Pattern 1: Change Brand Colors**
```css
/* Find and replace these values throughout the CSS */
#0066cc → #YOUR_PRIMARY_COLOR
#003d7a → #YOUR_SECONDARY_COLOR
```

**Pattern 2: Add More Features**
```html
<!-- Duplicate a feature-card div -->
<div class="feature-card">
  <div class="feature-icon">🆕</div>
  <h3>New Feature</h3>
  <p>Description</p>
  <a href="/docs/features/new/">Learn more →</a>
</div>
```

**Pattern 3: Adjust Section Order**
```html
<!-- Move sections by cutting/pasting entire <section> blocks -->
<!-- Example: Move Quick Start before Key Features -->
<section class="quick-start">...</section>
<section class="key-features">...</section>
```

**Pattern 4: Remove Sections**
```html
<!-- Simply delete the entire <section> you don't need -->
<!-- Don't forget to remove associated CSS if unused elsewhere -->
```

#### 3.5.9 Maintenance Notes

**When to update custom homepage:**
- Major new features released
- Branding/colors change
- User feedback indicates confusion
- Metrics show high bounce rate
- Competitive positioning shifts

**Version control tip:**
```bash
# Document major homepage changes in commit messages
git commit -m "Homepage: Add new AI features section"
```

**Analytics recommendations:**
- Track button click rates (Get Started vs Explore Features)
- Monitor scroll depth (which sections are seen)
- A/B test button text or section order
- Track mobile vs desktop engagement

---

### Phase 4: Deployment Setup

#### 4.1 Create GitHub Actions Workflow

Create `.github/workflows/deploy-docs.yml`:

**Option A: Deploy to Separate Repository (Recommended for custom domains)**

```yaml
name: Deploy Hugo Documentation to Docs Repository

on:
  push:
    branches:
      - main
    paths:
      - 'docs/learn/**'
      - '.github/workflows/deploy-docs.yml'
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN
permissions:
  contents: write

# Allow only one concurrent deployment
concurrency:
  group: "docs-deploy"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build and deploy job
  build-and-deploy:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.153.2
      TARGET_REPO: {owner}/{app-name}-docs
      TARGET_BRANCH: main
    steps:
      - name: Checkout source repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.21'

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: ${{ env.HUGO_VERSION }}
          extended: true

      - name: Download Hugo modules
        working-directory: docs/learn
        run: hugo mod get

      - name: Build with Hugo
        working-directory: docs/learn
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: America/Los_Angeles
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "https://docs.example.com/"

      - name: Deploy to docs repository
        uses: peaceiris/actions-gh-pages@v4
        with:
          personal_token: ${{ secrets.DOCS_DEPLOY_TOKEN }}
          external_repository: {owner}/{app-name}-docs
          publish_branch: main
          publish_dir: ./docs/learn/public
          user_name: 'github-actions[bot]'
          user_email: 'github-actions[bot]@users.noreply.github.com'
          commit_message: 'Deploy documentation from ${{ github.sha }}'
```

**Option B: Deploy to Same Repository GitHub Pages**

```yaml
name: Deploy Documentation to GitHub Pages

on:
  push:
    branches:
      - main
    paths:
      - 'docs/learn/**'
      - '.github/workflows/deploy-docs.yml'
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.153.2
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.21'

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: ${{ env.HUGO_VERSION }}
          extended: true

      - name: Download Hugo modules
        working-directory: docs/learn
        run: hugo mod get

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5

      - name: Build with Hugo
        working-directory: docs/learn
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./docs/learn/public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**Key Differences:**
- **Option A**: Deploys to separate `{app-name}-docs` repository, requires PAT token, better for custom domains
- **Option B**: Deploys to same repository's GitHub Pages, uses default token, simpler setup
- **Both options**: Include Go setup and `hugo mod get` for Hugo modules support

#### 4.2 Configure GitHub Personal Access Token (For External Repository Deployment)

**Only required if using Option A (separate repository) workflow**

1. **Create Fine-Grained Personal Access Token:**
   - Go to GitHub Settings → Developer settings → Personal access tokens → Fine-grained tokens
   - Generate new token with:
     - Resource owner: Your account
     - Repository access: Only select the docs repository
     - Repository permissions: Contents (Read and write)
     - Expiration: 90 days or longer

2. **Add Token as Repository Secret:**
   - Go to source repository Settings → Secrets and variables → Actions
   - Create new secret:
     - Name: `DOCS_DEPLOY_TOKEN`
     - Value: Paste the token

3. **Verify Token Works:**
   - Trigger the workflow
   - Check for successful deployment to external repository

**Security Notes:**
- Never commit tokens to repository
- Use fine-grained tokens with minimal permissions
- Set reasonable expiration dates
- Rotate tokens before expiration

#### 4.3 Configure Custom Domain

Create `static/CNAME`:
```
docs.example.com
```

#### 4.4 Add Supporting Files

**Create `.gitignore`:**
```
# Hugo build output
public/
resources/
.hugo_build.lock

# Development
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# Temporary files
*.tmp
*.bak
*.log

# Node modules
node_modules/
```

**Create `README.md` in docs/learn:**
```markdown
# Application Name Documentation

This directory contains the Hugo-based documentation site.

## Local Development

```bash
cd docs/learn
hugo server -D
```

Open http://localhost:1313

## Building

```bash
hugo --gc --minify
```

## Deployment

Automatically deployed to GitHub Pages via GitHub Actions.
```

---

### Phase 5: Testing & Quality Assurance

#### 5.1 Local Testing

**Build test:**
```bash
cd docs/learn
hugo --gc --minify
```

**Check for:**
- ✅ No build errors
- ✅ All pages generate
- ✅ Links are valid
- ✅ Images load correctly

**Development server:**
```bash
hugo server -D
```

**Test:**
- ✅ Navigation works
- ✅ Search functions (if enabled)
- ✅ Mobile responsive
- ✅ All screenshots display
- ✅ Internal links work
- ✅ External links work

#### 5.2 Content Review

**Checklist:**
- [ ] All features documented
- [ ] Screenshots in place (or placeholders clear)
- [ ] Code examples tested
- [ ] No broken links
- [ ] Consistent formatting
- [ ] Spell check completed
- [ ] Technical accuracy verified
- [ ] Clear navigation path

#### 5.3 Cross-Browser Testing

Test on:
- Chrome/Edge (Chromium)
- Firefox
- Safari
- Mobile browsers (iOS Safari, Chrome Android)

---

### Phase 6: Deployment

#### 6.1 DNS Configuration

**For custom domain:**
1. Add CNAME record in DNS provider
   ```
   Type: CNAME
   Name: docs (or subdomain name)
   Value: {username}.github.io
   TTL: 3600
   ```
2. Wait for DNS propagation (5-60 minutes)
3. Verify with `dig docs.example.com`

#### 6.2 GitHub Repository Setup

**Option A: Separate Repository (Recommended for custom domains)**
1. Create new public repository: `{app-name}-docs`
2. **Leave it empty** (no README, no initialization)
3. Configure GitHub Pages in docs repo:
   - Settings → Pages
   - Source: Deploy from branch `main` / `root`
   - Add custom domain if desired
4. Documentation will be auto-deployed by workflow from source repo

**Option B: Same Repository**
- Documentation in `docs/learn/` directory
- Workflow deploys to same repository's GitHub Pages
- Configure: Settings → Pages → Source: GitHub Actions
- Simpler but less flexible for custom domains

#### 6.3 GitHub Pages Configuration

1. Go to repository Settings → Pages
2. Source: **GitHub Actions**
3. Custom domain: Enter your domain
4. Wait for DNS check
5. Enable "Enforce HTTPS"

#### 6.4 First Deployment

```bash
git add docs/learn/
git add .github/workflows/deploy-docs.yml
git commit -m "Add documentation site"
git push origin main
```

Monitor in Actions tab.

---

## Deliverables Checklist

### For Stakeholder

- [ ] Live documentation site URL
- [ ] Screenshot capture guide
- [ ] Deployment checklist
- [ ] Maintenance instructions
- [ ] Access to GitHub repository

### Documentation Assets

- [ ] Complete Hugo site structure
- [ ] All content pages (with placeholders if screenshots pending)
- [ ] GitHub Actions workflow
- [ ] Configuration files
- [ ] README and setup guides
- [ ] Screenshot reference guide
- [ ] Deployment checklist

---

## Reusable Templates

### Template 1: Home Page

```markdown
---
title: "Application Name"
description: "Brief app description"
---

# Application Name Documentation

## Brief tagline about what the app does

[2-3 sentence overview]

---

## Key Features

### 🎯 **Feature 1**
Brief description  
[Learn more →](/docs/features/feature1/)

### 🔧 **Feature 2**
Brief description  
[Learn more →](/docs/features/feature2/)

[... more features ...]

---

## Why This Application?

### 🔍 Benefit 1
Description

### 🚀 Benefit 2
Description

### 🎯 Benefit 3
Description

---

## Quick Start

### 1. Install
Brief installation instruction

### 2. Configure
Brief configuration instruction

### 3. Run
Brief run instruction

---

**Ready to get started?** Head to the [Getting Started Guide](/docs/getting-started/)
```

### Template 2: Feature Page

See "Content Structure Template" in Phase 3.1 above.

### Template 3: Getting Started Page

```markdown
---
title: "Installation"
description: "How to install and run Application Name"
weight: 1
---

## Prerequisites

Before you begin, ensure you have:
- Requirement 1
- Requirement 2
- Requirement 3

---

## Installation Methods

### Method 1: From Source

1. Step 1
2. Step 2
3. Step 3

### Method 2: Docker

1. Step 1
2. Step 2

---

## Verify Installation

[How to verify it's working]

---

## Next Steps

- [Create Your First X](/docs/getting-started/quickstart/)
- [Understand Basic Concepts](/docs/getting-started/concepts/)
```

---

## Common Pitfalls & Solutions

### Problem: Git Submodules Error in GitHub Actions

**Symptom**: `fatal: No url found for submodule path 'docs/learn/themes/...' in .gitmodules`

**Solutions:**
1. **Remove** `submodules: recursive` from checkout action (theme should use Hugo modules, not git submodules)
2. **Add** Go setup step: `actions/setup-go@v5`
3. **Add** Hugo modules download step: `hugo mod get` before building
4. Verify `hugo.yaml` uses `module.imports` not git submodules

### Problem: Theme Shortcodes Not Working

**Symptom**: Build fails with "shortcode not found" errors

**Solutions:**
1. Ensure Hugo modules are initialized: `hugo mod init`
2. Run `hugo mod get` and `hugo mod tidy`
3. Check theme imports in hugo.yaml
4. Alternative: Convert shortcodes to plain markdown

### Problem: Images Not Displaying

**Symptom**: Broken image links on deployed site

**Solutions:**
1. Verify images are in `static/` directory
2. Check filenames match exactly (case-sensitive)
3. Ensure files were committed and pushed
4. Check image paths (no leading `/` needed for static files)

### Problem: Custom Domain Not Working

**Symptom**: 404 or domain doesn't resolve

**Solutions:**
1. Verify CNAME file exists in `static/CNAME`
2. Check DNS CNAME record points to `{username}.github.io`
3. Wait for DNS propagation (up to 24 hours)
4. Verify custom domain set in GitHub Pages settings
5. Check domain doesn't have conflicting records

### Problem: Permission Denied When Deploying to External Repository

**Symptom**: `remote: Permission to {owner}/{repo}.git denied to github-actions[bot]`

**Solutions:**
1. Verify you're using `personal_token` not `github_token` in workflow
2. Create Fine-Grained Personal Access Token with correct permissions
3. Add token as `DOCS_DEPLOY_TOKEN` secret in **source** repository (not docs repo)
4. Ensure token has "Contents: Read and write" permission
5. Verify token is scoped to the correct target repository
6. Check token hasn't expired

### Problem: Build Succeeds Locally, Fails in Actions

**Symptom**: Local build works, GitHub Actions fails

**Solutions:**
1. Check Hugo version matches in workflow and locally
2. Verify all dependencies are available in Actions (especially Go for Hugo modules)
3. Add `hugo mod get` step before build if using Hugo modules
4. Remove `submodules: recursive` if theme uses Hugo modules
5. Review build logs for specific errors

---

## Best Practices

### Content Organization

1. **Logical hierarchy**: Getting Started → Features → Advanced → Reference
2. **Consistent naming**: Use lowercase with hyphens for files
3. **Weight parameter**: Control page order explicitly
4. **Progressive disclosure**: Simple → Complex information flow
5. **Cross-linking**: Link related pages liberally

### Writing Style

1. **Scannable**: Use headers, bullets, short paragraphs
2. **Action-oriented**: Focus on what users can do
3. **Examples first**: Show, then explain
4. **Error prevention**: Document common mistakes
5. **Visual hierarchy**: Use formatting consistently

### Maintenance

1. **Version docs**: Consider versioning for major releases
2. **Regular reviews**: Schedule quarterly content audits
3. **User feedback**: Add feedback mechanism
4. **Analytics**: Consider adding analytics to track usage
5. **Update workflow**: Document who updates what and when

---

## Customization Guide

### Adapting for Different Applications

**For CLI tools:**
- Focus heavily on command examples
- Include full command reference section
- Show common command combinations

**For APIs:**
- Add API reference with endpoint documentation
- Include request/response examples
- Authentication section is critical

**For Web applications:**
- Visual screenshots are essential
- Focus on user workflows
- Include troubleshooting for browser issues

**For Libraries/SDKs:**
- Code examples in multiple languages
- API documentation front and center
- Integration guides for popular frameworks

### Branding Customization

**Colors:**
```yaml
params:
  primary: "#your-primary-color"
  secondary: "#your-secondary-color"
```

**Logo:**
- Add logo files to `static/images/`
- Reference in hugo.yaml params
- Provide both light and dark versions

**Custom CSS:**
- Add to `assets/css/custom.css`
- Reference in hugo.yaml

---

## Time Estimates

**For a typical application:**

- **Planning & Discovery**: 2-4 hours
- **Site Setup**: 1-2 hours
- **Content Creation** (without screenshots): 6-12 hours
- **Screenshot Capture**: 3-8 hours
- **Deployment Setup**: 1-2 hours
- **Testing & QA**: 2-4 hours

**Total: 15-32 hours** depending on:
- Number of features (affects content creation time)
- Screenshot count (major time factor)
- Complexity of features
- Existing documentation quality
- Stakeholder responsiveness

---

## Success Metrics

### Immediate Success

- [ ] Site builds without errors
- [ ] All pages accessible
- [ ] Navigation works correctly
- [ ] Screenshots display properly
- [ ] Deployed to custom domain
- [ ] HTTPS enabled

### Long-term Success

- [ ] User feedback positive
- [ ] Support requests decrease
- [ ] Documentation referenced in issues
- [ ] Search works effectively
- [ ] Mobile traffic supported
- [ ] Regular content updates maintained

---

## Tools & Resources

### Recommended Tools

- **Hugo**: https://gohugo.io/
- **Lotus Docs Theme**: https://github.com/colinwilson/lotusdocs
- **Alternative Themes**:
  - Docsy: https://www.docsy.dev/
  - Book: https://github.com/alex-shpak/hugo-book
  - Learn: https://github.com/matcornic/hugo-theme-learn

### Useful Links

- Hugo Documentation: https://gohugo.io/documentation/
- Markdown Guide: https://www.markdownguide.org/
- GitHub Pages Docs: https://docs.github.com/en/pages
- DNS Checker: https://dnschecker.org/

### Screenshot Tools

- **macOS**: Built-in Screenshot (Cmd+Shift+4)
- **Windows**: Snipping Tool or Snip & Sketch
- **Linux**: GNOME Screenshot, Flameshot
- **Browser Extensions**: Nimbus Screenshot, Awesome Screenshot

---

## Next Steps After Documentation is Live

1. **Announce**: Share documentation URL with users
2. **Link**: Add prominent link from main repository README
3. **Monitor**: Watch for 404s and broken links
4. **Gather feedback**: Add feedback mechanism
5. **Iterate**: Schedule regular content reviews
6. **Expand**: Add advanced topics based on user questions
7. **Maintain**: Keep screenshots and examples current

---

## Conclusion

This skill provides a repeatable, professional approach to creating comprehensive documentation sites. By following this methodology, you can transform any application from basic README documentation into a polished, user-friendly learning experience.

**Key Takeaways:**
- Plan thoroughly before writing
- Use consistent structure and templates
- Prioritize visual documentation (screenshots)
- Automate deployment with CI/CD
- Test extensively before launch
- Maintain regularly after launch

This approach works for applications of any size, from small utilities to enterprise platforms. Adjust the level of detail and number of pages based on your application's complexity and user needs.
