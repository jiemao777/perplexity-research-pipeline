---
name: perplexity-research-pipeline
description: Use Perplexity in the browser to run deep research from a user-provided prompt, especially when the user explicitly asks to use Perplexity, says "deep search/deep research", wants browser-based Perplexity instead of native web search, or wants a research-to-content pipeline. Also use when the user wants to: (1) generate a research Markdown file from Perplexity/browser research, (2) treat that Markdown as the information section of a content-generation workflow, (3) assemble a final writing prompt from a content-generation template plus research findings. This skill uses web-access CDP for browser operations. The pipeline stops after assembling the final prompt — the user manually copies it to ChatGPT for generation.
---

# Perplexity Browser Search

Use this skill when the user specifically wants Perplexity in the browser, not a normal web search, or when the user wants a full browser-driven research-to-writing pipeline.

**依赖：** 浏览器操作通过 `web-access` skill 的 CDP Proxy 实现。执行前需确保 Chrome 已启用远程调试。

---

## Browser CDP Setup

Before browser operations, verify CDP availability:

```bash
bash ~/.claude/skills/web-access/scripts/check-deps.sh
```

Requirements:
- **Node.js 22+**
- **Chrome remote-debugging**: Open `chrome://inspect/#remote-debugging` in Chrome, enable **"Allow remote debugging for this browser instance"**, then restart Chrome.

If CDP is not ready, guide the user to complete setup before proceeding.

## Core operating modes

Support these two modes:

1. **Research-only mode**
   - Run Perplexity research.
   - Extract the final answer body.
   - Return it directly or save it as Markdown.

2. **Research-to-prompt mode (Pangoo pipeline)**
   - Run Perplexity research.
   - Save the research output as a Markdown or text file.
   - Treat that file as the `information` section for a content-generation prompt.
   - Support two research prompt families: one for cluster pages and one for pillar pages.
   - Fill the rest of the prompt from the user's content-generation template.
   - Save the assembled final prompt as Markdown.
   - **Stop here.** Notify the user the final prompt is ready and provide the file path.
   - The user manually copies the prompt to ChatGPT for article generation.

3. **Batch override mode (explicit user request only)**
   - Use this only when the user explicitly asks to finish multiple remaining keywords in one pass without waiting for manual URL callbacks after each article.
   - In this mode, generate predicted final article URLs from the slugs, update internal links immediately, and continue through the whole batch.
   - Make it clear this is an override of the skill's default one-by-one publishing flow.

Default to research-only mode unless the user clearly asks for the full pipeline.

---

## Pangoo content-generation prompt template

This is the fixed template the user provides. Use it as-is for every article. Only the `<information>`, `<keyword>`, `<internal links>`, and `<image embeds>` sections change per article.

```
1) Core Persona
Identity: You are an expert SEO + NLP content writer and a true specialist in the given topic.
Objective: Write natural, non-robotic content with expert insights, practical tips, and light real-world anecdotes.
Audience: Global importers / private-label operators / China sourcing buyers (B2B).

2) Writing Style & Tone
- Use simple words a 7-year-old can understand.
- Use Subject-Verb-Object sentences.
- Use active voice.
- Be clear and concise. Remove filler.
- Avoid complex or overly formal sentences.
- Avoid cringey or childish analogies.

3) Article Structure & Formatting (Final Output Must Follow This)
A) Start with a "Key Takeaways" table OR bullet list at the very top.
B) Then write the full article body with 10–15 H2 headings (##).
 - Each H2 section: ~120–170 words.
 - Use Markdown formatting: lists, tables, bold, short paragraphs.
C) End with a "Frequently Asked Questions" (FAQ) section at the very bottom.

4) SEO & Technical Requirements
4.1 Internal Linking (IMPORTANT)
- You must use the URLs provided in <internal links>.
- Integrate links naturally inside relevant sentences; anchors must be keyword-rich.
- Avoid "click here".
- Keep link relevance high(max 2-3 links per section)
- HARD RULE: Never create a standalone "link list / link map" section.
 (No dumping links in one block. No "Link Map". No "Resources list" that is only links.)
- If the list is very long:
 - Spread links across the article body AND the FAQ answers.
 - You may use short bulleted lists ONLY when each bullet contains 1 helpful sentence (not just a link).

4.2 Image Embedding (IMPORTANT)
- Use images from <image embeds>.
- Embed images naturally in the article where relevant (aim for frequent use, but don't force it).
- Format:
 [![Descriptive alt text with keywords](image_URL)](link_URL)
- Every image must be clickable and point to a relevant destination link.
- Alt text rules:
 - Include the primary keyword when natural
 - Describe what is in the image
 - Under 125 characters

5) Prohibitions (What to Avoid)
- Do NOT break the fourth wall: no "in this article", no "the image above", no "we will discuss".
- Do NOT use prohibited marketing/AI language:
 step into, believe it or not, buckle up, in additional, additionally, navigating, when it comes to, embarking, embark, bespoke,
 look no further, however it is important to note, meticulous, meticulously, complexities, realm, tailored, towards, underpins,
 everchanging, ever-evolving, the world of, not only, diving into, seeking more than just, designed to enhance, it's not merely,
 our suite, it is advisable, daunting, dives, dive in, let's delve, let's dive in, in the heart of, remember, in an era,
 picture this, amongst, unlock the secrets, unveil the secrets, robust.

6) Internal Workflow (INTERNAL ONLY — DO NOT OUTPUT)
- First: Build an internal outline using <internal links> and <image embeds>.
- Second: Build an internal link placement plan (internal link map) to ensure coverage.
- Third: Write the final article.
- Final check: Confirm you did NOT output outline/link map/workflow, and that all rules are met.

7) Inputs
<information>
[research content goes here]
</information>

<keyword>
[keywords go here]
</keyword>

<internal links>
[URLs go here, one per line]
</internal links>

<image embeds>
[markdown image embeds go here, one per line]
</image embeds>

8) Final Reminder
Your response must start directly with the "Key Takeaways".
No preface. No outline. No link map. No meta text.
ONLY the article.
```

---

## Research prompt templates

### Cluster page research prompt

```
I am writing an article for www.pangoo.biz. I want you to do some research on the blog post topic cluster of "[CLUSTER KEYWORD]" for the pillar page "[PILLAR KEYWORD]" but the context of the business into account and including data. Also what are the feedbacks from twitter users and reddit.com?
```

### Pillar page research prompt

```
I am writing an article for www.pangoo.biz. I want you to do some research on the blog post pillar page of "[PILLAR KEYWORD]" and include the Topic Cluster "[CLUSTER KEYWORD 1]" "[CLUSTER KEYWORD 2]" "[CLUSTER KEYWORD 3]" "[CLUSTER KEYWORD 4]" "[CLUSTER KEYWORD 5]" but the context of the business into account and including data. Also what are the feedbacks from twitter users and reddit.com?
```

---

## Pangoo content sequencing

Default order for a pillar + cluster set:

1. cluster page 1
2. cluster page 2
3. cluster page 3
4. cluster page 4
5. cluster page 5
6. pillar page

Use the finished cluster set as input context for the pillar-page research and writing steps.

---

## Complete pipeline step-by-step

This is the exact order to follow for each article. Every step has been tested and refined through real runs.

### Step 1: Perplexity Research (CDP)

**前置检查：**
```bash
bash ~/.claude/skills/web-access/scripts/check-deps.sh
```
确保 CDP Proxy 已连接。

**操作流程：**

1. **构建 Perplexity 搜索 URL**
   - 对 cluster 或 pillar research prompt 进行 URL 编码
   - 格式：`https://www.perplexity.ai/search?q=<URL_ENCODED_PROMPT>`

2. **打开 Perplexity 页面**
   ```bash
   curl -s "http://localhost:3456/new?url=https://www.perplexity.ai/search?q=<ENCODED_PROMPT>"
   ```
   - 返回 `targetId`，用于后续操作
   - 页面会自动等待初始加载完成

3. **等待答案渲染**
   - Perplexity 答案需要 30–60 秒完整渲染
   - 使用短等待（5–8 秒）+ 循环检查，避免长等待导致连接丢失
   ```bash
   # 检查答案是否已渲染
   curl -s -X POST "http://localhost:3456/eval?target=TARGET_ID" -d '
     document.querySelector("[data-testid=\"answer-content\"]") ? "ready" : "loading"
   '
   ```

4. **提取答案内容**
   ```bash
   curl -s -X POST "http://localhost:3456/eval?target=TARGET_ID" -d '
     document.querySelector("[data-testid=\"answer-content\"]")?.innerText || document.body.innerText
   '
   ```
   - 如果返回为空，检查页面是否仍在加载或被 Cloudflare 拦截

5. **处理 Cloudflare 验证**
   - 如果检测到 Cloudflare 挑战页面，**暂停并请求用户手动完成验证**
   - 用户完成后，继续提取内容

6. **关闭 Tab**
   ```bash
   curl -s "http://localhost:3456/close?target=TARGET_ID"
   ```

**经验教训：**
- 直接 URL 搜索比在输入框输入更可靠
- Cloudflare 会阻止自动化访问，随时准备请求用户手动验证
- 使用短等待循环检查，避免单次长等待
- `curl /new` 返回的 `targetId` 是后续所有操作的标识
- 提取内容时优先使用具体的 CSS 选择器，而非全文提取

### Step 2: Save Research File

1. Clean the extracted Perplexity output: remove UI elements, source buttons, and follow-up question suggestions.
2. Structure the research into clear sections:
   - Topic and pillar context
   - Key findings with data points
   - Twitter/Reddit sentiment summary
   - Sources cited
3. Save as `topic-slug-research.md`.

### Step 3: Assemble Final Prompt

1. Take the content-generation prompt template from section "Pangoo content-generation prompt template" above.
2. Fill in `<information>` with the cleaned research content.
3. Fill in `<keyword>` with 3–8 relevant keywords including the primary target keyword.
4. Fill in `<internal links>` with the full available Pangoo internal URL list (from the master list or user-provided list). This is the default behavior.
5. Fill in `<image embeds>` with the full available image embed pool from the active image library file. This is the default behavior.
   - Only curate or trim the image list if the user explicitly asks for a smaller subset.
6. Save as `topic-slug-final-prompt.md`.

**Lessons learned:**
- Do not truncate the internal links list. ChatGPT handles long lists well and needs them to spread links across the article.
- For this Pangoo pipeline, default to full internal links.
- For this Pangoo pipeline, default to full image embeds.
- Assume downstream tooling may choose the best images, so the prompt should preserve the full candidate pool unless the user explicitly asks to trim it.
- Include a mix of product pages, guide pages, category pages, and trust pages (about-us, COA, MSDS, factory).
- The full prompt including all links and images can be large. This is fine.

### Step 4: Generate Article via ChatGPT (Optional - User Choice)

**User Choice:** Before proceeding, ask the user:
"文章生成方式：
1. 自动化：通过浏览器自动操作 ChatGPT 生成文章（需要已登录 ChatGPT）
2. 手动：打开 final prompt 文件，手动复制到 ChatGPT 生成

请选择 1 或 2："

**If user chooses Option 1 (Automatic):**

1. **Read final prompt content**
   ```bash
   # Read the final prompt file
   PROMPT_CONTENT=$(cat "C:/Users/Chen/slug-final-prompt.md")
   ```

2. **Open ChatGPT**
   ```bash
   curl -s "http://localhost:3456/new?url=https://chat.openai.com/"
   # Returns targetId
   ```

3. **Wait for page load**
   ```bash
   sleep 5
   ```

4. **Check login status**
   ```bash
   curl -s -X POST "http://localhost:3456/eval?target=TARGET_ID" -d '
     (() => {
       const textarea = document.querySelector("textarea[placeholder*=\"Message\"]") ||
                        document.querySelector("#prompt-textarea");
       return textarea ? "ready" : "loading";
     })()'
   ```

5. **Paste prompt and send**
   ```bash
   # Find textarea and paste content
   curl -s -X POST "http://localhost:3456/eval?target=TARGET_ID" -d "
     (() => {
       const textarea = document.querySelector('textarea[placeholder*=\"Message\"]') ||
                        document.querySelector('#prompt-textarea');
       if (textarea) {
         textarea.value = \`$PROMPT_CONTENT\`;
         textarea.dispatchEvent(new Event('input', { bubbles: true }));
         const sendBtn = document.querySelector('button[data-testid=\"send-button\"]') ||
                        document.querySelector('button[aria-label=\"Send message\"]');
         if (sendBtn && !sendBtn.disabled) {
           sendBtn.click();
           return 'sent';
         }
       }
       return 'error';
     })()"
   ```

6. **Wait for generation to complete** (3-5 minutes typically)
   - Use short wait loops (15-20 seconds) with status checks
   - Check for completion indicators (stop button disappears, final response visible)
   ```bash
   # Loop until generation completes
   for i in {1..20}; do
     STATUS=$(curl -s -X POST "http://localhost:3456/eval?target=TARGET_ID" -d '
       (() => {
         const stopBtn = document.querySelector("button[aria-label=\"Stop generating\"]");
         const responses = document.querySelectorAll("[data-message-author-role=\"assistant\"]");
         return stopBtn ? "generating" : (responses.length > 0 ? "complete" : "waiting");
       })()')

     if [ "$STATUS" = "complete" ]; then
       break
     fi
     sleep 15
   done
   ```

7. **Extract generated article**
   ```bash
   curl -s -X POST "http://localhost:3456/eval?target=TARGET_ID" -d '
     (() => {
       const responses = document.querySelectorAll("[data-message-author-role=\"assistant\"]");
       const lastResponse = responses[responses.length - 1];
       return lastResponse ? lastResponse.innerText : "";
     })()' > "C:/Users/Chen/slug-generated-article.md"
   ```

8. **Close ChatGPT tab**
   ```bash
   curl -s "http://localhost:3456/close?target=TARGET_ID"
   ```

9. **Open generated article file**
   ```bash
   start "" "C:/Users/Chen/slug-generated-article.md"
   ```

10. **Ask user to review and publish**
    - Confirm the generated article is ready
    - Request the published URL once live

**If user chooses Option 2 (Manual):**

1. Confirm the final prompt file has been saved.
2. Tell the user the file path and approximate file size.
3. **Automatically open the final prompt MD file** using the default system application:
   - Windows: `start "" "path/to/file.md"`
   - macOS/Linux: `open "path/to/file.md"`
4. Remind the user to manually copy the file content to ChatGPT and generate the article.
5. Wait for user to provide published article URL.

**Common Step 5 (Both Options):**

1. Ask the user for the published article URL once it is live.
2. When the user provides the URL, append it to the internal links source file (or update the active master links file) in the same format as existing entries.
3. **Automatically proceed to the next keyword** (if any remain in the batch).
4. Batch override exception: if the user explicitly asked for one-pass completion without waiting for manual URL callbacks, do not pause. Generate predicted final URLs from slugs, update internal links immediately, and finish the whole batch.

---

**Troubleshooting ChatGPT Automation:**

- **Not logged in:** Guide user to login to ChatGPT in browser first, then retry
- **Generation timeout:** Increase wait loops, article generation can take 3-5 minutes
- **Content truncated:** Check if ChatGPT hit token limit, may need to extract multiple response blocks
- **Send button disabled:** Wait a few seconds for ChatGPT to process pasted content before clicking

---

## File naming convention

Use a slug derived from the cluster/pillar keyword:

- `slug-research.md` — Perplexity research findings
- `slug-final-prompt.md` — Assembled prompt for ChatGPT

Example for "Piglet Gut Health & Post-Weaning Program":
- `piglet-gut-health-post-weaning-program-research.md`
- `piglet-gut-health-post-weaning-program-final-prompt.md`

---

## Image embeds library

Prefer a single master image library file for the current content batch whenever possible, for example `image-embeds-[topic]-master.txt`.
Only keep multiple image library files if the user explicitly wants separate pools.
When the user manually provides image sources in chat, treat that user-provided list as the authoritative source for the current batch and normalize it into the master image library file.

Maintain the active image library file in a structured format that contains all available images organized by category:

```
# Image Embeds - [Pillar Name]

## Products (amino acids, minerals, additives)
URL: ..., Image URL: ..., Title: ...

## Yeast Products
...

## Feed Proteins
...

## Packaging & Company
...

## Infographics
...
```

When converting the library to prompt-ready markdown image embeds:
- If the user asks for a curated subset, pick the most relevant set for the topic.
- If the user asks for the full pool because another downstream tool will choose the best images, include the entire valid deduplicated image list.
- Deduplicate by `(link URL, image URL)`.
- Exclude invalid items such as empty image URLs, obvious PDFs, or page URLs mistakenly used as image URLs.

Markdown format:
```
[![Descriptive alt text with keywords](image_URL)](link_URL)
```

---

## Tab management (CDP)

- **Always close Perplexity tabs** after successful extraction using `curl /close`
- Keep the `targetId` from `curl /new` for all operations
- Task completion = close the tab you created
- CDP Proxy 持续运行，不建议主动停止

---

## Handling failures

### CDP Proxy not running
- Run `bash ~/.claude/skills/web-access/scripts/check-deps.sh`
- If Chrome not connected: guide user to `chrome://inspect/#remote-debugging`
- Restart Chrome if needed after enabling remote debugging

### Cloudflare on Perplexity
- Pause. Ask user to verify manually in the browser. Resume after.
- User's Chrome carries natural login state, may help bypass some checks

### Empty eval result
- Page may still be loading. Wait 5-8 seconds and retry.
- Use `/screenshot` to see what's actually on screen:
  ```bash
  curl -s "http://localhost:3456/screenshot?target=TARGET_ID&file=/tmp/perplexity.png"
  ```
- Then read the screenshot to diagnose

### Target lost / navigation failed
- If `targetId` no longer valid, list current tabs:
  ```bash
  curl -s http://localhost:3456/targets
  ```
- Re-locate the Perplexity tab or create a new one

### No offline fallback requested
- If the user has explicitly said not to use offline fallback, do not fabricate or synthesize the research stage from general knowledge.
- Either continue debugging the CDP path or ask the user to paste the Perplexity output manually.

---

## Good trigger examples

- "执行第X个关键词"
- "开始第X篇 cluster page"
- "先做 research，然后生成 prompt"
- "用 Perplexity 深度研究这个 topic"
- "帮我做成 research → prompt 的流水线"

---

## Final note

This skill runs Perplexity browser research and assembles a final ChatGPT prompt. The pipeline stops after saving the final prompt — the user copies it to ChatGPT manually. This saves significant tokens by avoiding long browser interaction loops for ChatGPT generation and extraction.
