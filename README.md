# Perplexity Research Pipeline

A powerful Claude Code skill that automates research-to-content workflows using Perplexity AI and browser automation. Perfect for SEO content creators, researchers, and anyone who needs systematic, deep research before writing.

## 🚀 Features

- **Automated Perplexity Research**: Uses browser automation to run deep research with Twitter/Reddit sentiment analysis
- **Research-to-Prompt Pipeline**: Automatically converts research findings into optimized writing prompts
- **CDP Browser Control**: Leverages Chrome DevTools Protocol for reliable browser operations
- **Smart Content Generation**: Integrates with ChatGPT for high-quality, SEO-optimized content
- **Batch Processing**: Supports multi-keyword workflows with automatic progression
- **Project Organization**: Auto-creates project folders with structured file management

## 📋 Requirements

- **Node.js 22+** - Required for URL encoding and CDP operations
- **Chrome with Remote Debugging** - Enable at `chrome://inspect/#remote-debugging`
- **Claude Code** - With `web-access` skill installed
- **CDP Proxy** - Started via `web-access` skill

## 🔧 Installation

### Step 1: Install Dependencies

```bash
# Verify Node.js installation
node --version  # Should be v22+

# Enable Chrome Remote Debugging
# 1. Open chrome://inspect/#remote-debugging
# 2. Enable "Allow remote debugging for this browser instance"
# 3. Restart Chrome
```

### Step 2: Install Web-Access Skill

```bash
# Install web-access skill (if not already installed)
# This skill depends on web-access for CDP browser control
```

### Step 3: Install This Skill

```bash
# Clone to Claude Code skills directory
cd ~/.claude/skills
git clone https://github.com/jiemao777/perplexity-research-pipeline.git

# Or manually copy the folder
# Copy the entire folder to ~/.claude/skills/
```

## 📖 Usage

### Basic Research Mode

Simply ask Claude Code to use Perplexity for research:

```
"Use Perplexity to research the latest trends in electric vehicle batteries"
```

### Research-to-Prompt Pipeline (SEO Content)

For content creation workflows:

```
"Create a research-to-prompt pipeline for 'bulk feed additives supplier China' with cluster keywords:
1. import bulk feed additives from China
2. feed additives manufacturing process
3. bulk feed additives pricing and MOQ
4. global feed additives markets
5. FAQ for international buyers"
```

### Batch Processing

```
"Execute all 5 cluster keywords for the amino acid project using Perplexity research"
```

## 🔄 Workflow

This skill follows a systematic 3-step process:

### Step 1: Perplexity Research
- Opens Perplexity in automated browser
- Runs deep research with your custom prompt
- Includes Twitter/Reddit sentiment analysis
- Extracts comprehensive findings

### Step 2: Research File Creation
- Saves cleaned research as Markdown
- Structures content with sections:
  - Topic and context
  - Key findings with data
  - Social media sentiment
  - Sources cited

### Step 3: Final Prompt Assembly
- Combines research with content-generation template
- Fills in keywords, internal links, image embeds
- Produces ChatGPT-ready prompt file
- Stops for manual review (you control final generation)

## 📁 Project Structure

```
your-project-folder/
├── keyword-1-research.md
├── keyword-1-final-prompt.md
├── keyword-2-research.md
├── keyword-2-final-prompt.md
├── progress.txt
└── pillar-research.md
    └── pillar-final-prompt.md
```

## ⚙️ Configuration

### Content-Generation Template

The skill uses a fixed template structure (defined in `skill.md`). Key sections:

1. **Core Persona** - Expert SEO + NLP writer identity
2. **Writing Style** - Simple, clear, active voice
3. **Article Structure** - Key Takeaways + 10-15 H2 headings + FAQ
4. **SEO Requirements** - Internal linking, image embedding
5. **Prohibitions** - Avoids AI/marketing fluff language

### Customization Points

You can customize per article:
- `<information>` - Research content from Perplexity
- `<keyword>` - Target keywords (3-8 related keywords)
- `<internal links>` - Full URL list for internal linking
- `<image embeds>` - Image library with structured metadata

## 🎯 Best Practices

### 1. Research Prompt Design

**For Cluster Pages:**
```
I am writing an article for www.yoursite.com. I want you to do some research on the blog post topic cluster of "[CLUSTER KEYWORD]" for the pillar page "[PILLAR KEYWORD]" but the context of the business into account and including data. Also what are the feedbacks from twitter users and reddit.com?
```

**For Pillar Pages:**
```
I am writing an article for www.yoursite.com. I want you to do some research on the blog post pillar page of "[PILLAR KEYWORD]" and include the Topic Cluster "[CLUSTER 1]" "[CLUSTER 2]" "[CLUSTER 3]" "[CLUSTER 4]" "[CLUSTER 5]" but the context of the business into account and including data. Also what are the feedbacks from twitter users and reddit.com?
```

### 2. File Naming

Use descriptive slugs derived from keywords:
- `l-lysine-feed-grade-research.md`
- `dl-methionine-feed-grade-final-prompt.md`
- `amino-acid-supplier-lysine-methionine-research.md`

### 3. Content Sequencing

Recommended order for pillar + cluster sets:
1. Cluster page 1
2. Cluster page 2
3. Cluster page 3
4. Cluster page 4
5. Cluster page 5
6. Pillar page (uses all 5 clusters as context)

## 🔍 Troubleshooting

### CDP Proxy Not Running

```bash
bash ~/.claude/skills/web-access/scripts/check-deps.sh
```

### Chrome Not Connected

1. Open `chrome://inspect/#remote-debugging`
2. Enable "Allow remote debugging for this browser instance"
3. Restart Chrome

### Cloudflare Blocking

- Perplexity may trigger Cloudflare challenges
- Pause and complete verification manually in browser
- Resume after verification

### Target Lost/Navigation Failed

```bash
# List current tabs
curl -s http://localhost:3456/targets

# Re-create Perplexity tab
curl -s "http://localhost:3456/new?url=https://www.perplexity.ai/search?q=<ENCODED_PROMPT>"
```

## 📊 Example Output

### Research File Structure

```markdown
# Research: Your Topic Here

## Topic and Context
**Cluster Keyword:** your keyword here
**Pillar Keyword:** main pillar keyword
**Business Context:** www.yoursite.com

## Key Findings

### 1. Main Point
- Data point 1
- Data point 2

### 2. Another Point
- Supporting evidence

## Sources Referenced
1. Source A
2. Source B
```

### Final Prompt Structure

```markdown
1) Core Persona
Identity: Expert SEO + NLP content writer

2) Writing Style & Tone
- Use simple words
- Active voice
- Clear and concise

3) Article Structure
A) Key Takeaways
B) 10-15 H2 headings
C) FAQ section

4) SEO Requirements
- Internal linking strategy
- Image embedding rules

7) Inputs
<information>
[Research content here]
</information>

<keyword>
[Target keywords]
</keyword>

<internal links>
[URL list]
</internal links>

<image embeds>
[Image markdown]
</image embeds>
```

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📝 License

This skill is provided as-is for use with Claude Code. Feel free to adapt and modify for your needs.

## 🔗 Links

- **Repository**: https://github.com/jiemao777/perplexity-research-pipeline
- **Dependencies**: [web-access skill](https://github.com/your-repo/web-access)
- **Claude Code**: https://claude.ai/code

## 💡 Use Cases

- **SEO Content Creation**: Systematic keyword research + content generation
- **Market Research**: Deep dives with social sentiment analysis
- **Academic Research**: Comprehensive source gathering
- **Competitor Analysis**: Structured research workflows
- **Product Research**: Feature comparisons and user feedback

## 🎓 Advanced Features

### Batch Override Mode

For processing multiple keywords without manual URL callbacks:

```
"Finish all remaining keywords in one batch without waiting for published URLs"
```

This mode:
- Generates predicted URLs from slugs
- Updates internal links immediately
- Processes entire batch continuously

### Custom Image Libraries

Maintain master image files with structured format:

```
# Image Embeds - Pillar Name

## Products
URL: https://example.com/product, Image URL: https://example.com/image.jpg, Title: Product Name

## Packaging
URL: https://example.com/packaging, Image URL: https://example.com/package.jpg, Title: Packaging Options
```

Converts to markdown automatically:
```markdown
[![Product Name](https://example.com/image.jpg)](https://example.com/product)
```

---

**Made with ❤️ for researchers and content creators**

**Version**: 1.0.0
**Last Updated**: 2026-03-25
