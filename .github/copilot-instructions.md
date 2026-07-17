# Project Guidelines

## Generating, editing, or rendering BPMN diagrams

**Always use the `bpmn-mcp` MCP tools for bpmn and dmn file types — never write a Python/Node
script, a Graphviz/Mermaid workaround, or hand-roll BPMN XML.** The server
is registered as `bpmn-mcp` and
exposes tools like `start_session`, `add_task`, `connect`, `run_layout`,
`render_preview`/`render_image`, `describe_preview`, `finish`,
`load_bpmn_file`, `check_bpmn_file`, and `fix_bpmn_file`.

- "Generate/create a diagram from this text" → `start_session(text)`, then
  follow the stage-gated flow in [0-bpmn-mcp.md](../0-bpmn-mcp.md) through
  to `finish` (pass `outputPath` so the file is written directly).
- "Render/view/edit the diagram" (no new text given) → `load_bpmn_file`
  first (auto-discovers the single `*.bpmn` file if `path` is omitted),
  not `start_session`.
- "Find/fix errors in the diagram" → `check_bpmn_file` / `fix_bpmn_file`.
- Don't skip the review gates (`review_outline`/`review_structure`/
  `review_layout`) — they exist so the user approves structure before
  layout, and layout before the file is written.
- To verify the xml structure of a BPMN file, use `check_bpmn_file` rather than
  hand-inspecting the XML or running xmllint or other XML validation tools.
- To check the content of a BPMN file, use `describe_preview` rather than
  hand-inspecting the XML or running a script-based workaround.
- To export the preview image of a BPMN diagram to png or svg, use `render_image` rather than
  hand-inspecting the XML or running a script-based workaround.
- If the `bpmn-mcp` tools aren't available in the current chat session,
  say so and ask the user to enable the MCP server rather than
  substituting a script-based workaround and then `finish`ing the session.
- **ALWAYS use absolute file paths** for the `outputPath` parameter in
  `generate_diagram` and `generate_<type>_diagram` calls — relative paths
  may fail due to working directory ambiguity. Use the full path (e.g.
  `d:\diagram\bpmn-mcp\test\file.bpmn` on Windows or
  `/full/path/to/file.bpmn` on Unix).

## Generating, validating, or understanding Mermaid diagrams

**Always use the `mermaid-mcp` MCP tools for Mermaid diagrams — never write a
Python/Node script, hand-roll Mermaid syntax checks, or shell out to `mmdc`
or another CLI.** The server is registered as `mermaid-mcp` and exposes
`generate_diagram`, `validate_diagram`, and `parse_diagram`, covering all 8
supported diagram types: flowchart, classDiagram, sequenceDiagram,
stateDiagram(-v2), erDiagram, timeline, journey, and block-beta.

- "Generate/create a diagram" → compose the Mermaid diagram text yourself
  (you already know the syntax), then call `generate_diagram(content,
  outputPath)` to validate it and write the file — don't write the file
  directly with a generic file-write tool.
- **ALWAYS use absolute file paths** for the `outputPath` parameter in
  `generate_diagram` and `generate_<type>_diagram` calls — relative paths
  may fail due to working directory ambiguity. Use the full path (e.g.
  `d:\diagram\mermaid-mcp\test\file.mmd` on Windows or
  `/full/path/to/file.mmd` on Unix).
- "Check/validate this diagram" (without necessarily writing a file) →
  `validate_diagram`, passing either `content` or `path`.
- "Explain/understand what this diagram means" — for a diagram found in the
  repo, pasted by the user, or otherwise not authored by you in this
  conversation — → `parse_diagram`, passing either `content` or `path`; use
  its returned summary/narrative instead of hand-parsing the Mermaid text.
- `validate_diagram` and `parse_diagram` accept exactly one of `content` or
  `path` — don't pass both, and don't read the file yourself first just to
  pass its contents as `content` when `path` will do.
- The output file extension controls fencing automatically: `.md`/
  `.markdown` wraps the content in a ```` ```mermaid ```` fence, any other
  extension (e.g. `.mermaid`, `.mmd`) is written as raw Mermaid syntax with
  no fence — don't add or strip the fence yourself before calling
  `generate_diagram`.
- If `generate_diagram` or `validate_diagram` reports invalid content, fix
  the Mermaid syntax and retry rather than writing the file anyway or
  falling back to a script-based workaround.
- If the `mermaid-mcp` tools aren't available in the current chat session,
  say so and ask the user to enable the MCP server rather than
  substituting a script-based workaround.

### Generating a diagram from any existing description/spec file

The `mermaid-mcp` tools have no concept of a "description file" — they only
take `content` and `outputPath`, so this rule must be followed by whoever
calls them (i.e. by you, the assistant), not enforced by the server:

- **Whenever the diagram is being generated from a description/spec file that
  already exists in the workspace** (e.g. a `.txt`/`.md` requirements file the
  user points you at, not a diagram invented from a plain chat request with no
  backing file), the generated `.mmd`/`.mermaid`/`.md` output **always goes in
  the SAME folder as that source file**, using the **same basename** (just
  swap the extension) — never under `examples/` or any other unrelated
  folder, regardless of which subfolder the source file happens to live in.
- Before calling `generate_diagram`/`generate_<type>_diagram`, derive
  `outputPath` directly from the source file's own path (same directory,
  same basename, new extension) rather than picking a new location.
- This applies repo-wide, not just to `test/` — e.g. `test/NN-name.txt` spec
  files must produce `test/NN-name.mmd` (never elsewhere), and the same
  same-folder/same-basename rule applies to any other directory's spec files
  too.
- For files under `test/`, the two-digit `NN` prefix also indicates which
  diagram type milestone it corresponds to (see repo memory
  `/memories/repo/mermaid-mcp-parser.md` for the full type-addition history)
  — infer the intended diagram type from that prefix/filename when the spec
  text itself doesn't name a type.
- If there's no pre-existing source file (the user just describes a diagram
  in chat), this rule doesn't apply — ask the user for or infer a sensible
  output location as usual.
