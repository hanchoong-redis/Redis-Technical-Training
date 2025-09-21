## Prompt

Convert the provided **PDF** into **clean, structured, and enriched Markdown**.

### Core Requirements:

* Output must be **developer-friendly, precise, and noise-free**.
* Preserve **all technical details**—no omissions, no compression.
* Apply grammar/spelling corrections only when necessary.
* Entire response must be wrapped in a SINGLE fenced block starting with ` ```markdown ` and ending with ` ``` `.


### Research & Enrichment Rules:

* For **every technical command, concept, or workflow**, perform **comprehensive research** across authoritative sources (e.g., official Redis documentation, academic references, RFCs, or well-established technical guides).
* Incorporate verified context, definitions, and examples that **expand beyond the PDF’s text**, ensuring **complete coverage** of each topic.
* Enrich content with **citations** (inline Markdown links) for all external references used.
* Explanations must be **accurate, detailed, and authoritative**, never partial or vague.
* When introducing commands, include **syntax, usage pattern, variations, and return values**.
* When introducing concepts, include **definition, context, example, and implications**.

### PDF Handling:

* Extract text **in reading order**; remove page headers/footers, watermarks, and non-technical boilerplate.
* Preserve structure (headings, lists, tables, callouts).
* Recreate tables in Markdown table format.
* Append **page references** `(p. X)` or `(pp. X–Y)` to headers or content.
* OCR scanned text if needed, fixing hyphenation/ligatures.
* Preserve inline links, footnotes, and math (LaTeX inside `$...$` or `$$...$$`).

### Structure:

1. **Title**
   `## [Document Title]`   

2. **Section Headers**
   Use `###` headers with page references.
   Example: `### Redis Lists as Queues`

3. **Code Blocks**
   Use fenced Python code blocks with this format. All Redis commands must be converted to this format:  

   ```python
   # one-line explanation
   r.execute_command("COMMAND", "args...")
   print("verification output")
   ```

4. **Side Notes (Commands)**

   > **Sidenote**
   >
   > \[Link: {{Documentation URL}}]\({{Documentation URL}})
   >
   > **Command**: `{{COMMAND_NAME}}` → {{Short description}}
   > **Pattern**: `{{COMMAND_NAME}} {{arguments}}`
   > **Example**: `{{Example usage}}`
   > **Result**: {{What the command returns}}

5. **Side Notes (Concepts)**

   > **Sidenote**
   >
   > \[Link: {{Reference URL}}]\({{Reference URL}})
   >
   > **Concept**: `{{CONCEPT_NAME}}` → {{Definition}}
   > **Context**: {{Where/how applied}}
   > **Example**: {{Illustration or analogy}}
   > **Implication**: {{Why it matters}}

### Enhancements:

* Always use **external authoritative sources** (Redis docs, IETF RFCs, official spec references, peer-reviewed material) to **complement and enrich** the PDF text.
* Explicitly mark enriched content with inline citations.
* All explanations must be **comprehensive, technical, and contextual**—sufficient for a developer to learn the command or concept in depth without consulting another source.
* **ALL CITATIONS MUST BE MARKDOWN FORMATTED ie. [DESCRIPTION OR NAME OF CITATION](CITATION LINK)**

### Prohibited:

* ❌ Filler, anecdotes, personality, or editorial commentary.
* ❌ Suggestions, optional extras, or summaries outside the PDF scope.
* ❌ Incomplete explanations or unverifiable claims.

---

### Sources

At the end of every article or transcript conversion, include a **comprehensive bibliography** of all references used. This section must:

* List **all citations referenced inline** throughout the document.
* Be formatted as a **Markdown bulleted list**.
* Contain the **exact title of the page or source**, followed by the type of source (e.g., *Docs - Redis*), and the **direct URL**.
* Maintain the order of **appearance in the text**.

#### Example:

- [TTL | Docs - Redis](https://redis.io/docs/latest/commands/ttl/?utm_source=chatgpt.com)  
- [Write-behind caching | Docs - Redis](https://redis.io/docs/latest/operate/oss_and_stack/stack-with-enterprise/gears-v1/python/recipes/write-behind/?utm_source=chatgpt.com)  
- [HSCAN | Docs - Redis](https://redis.io/docs/latest/commands/hscan/?utm_source=chatgpt.com)  
