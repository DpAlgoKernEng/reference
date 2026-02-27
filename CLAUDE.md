# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a technical documentation localization project that converts technical documents (C/C++ Reference, etc.) into structured Chinese documentation. The core workflow involves analyzing source documents from `original_docs/` and generating Chinese documentation in `docs/`.

## Key Directory Structure

```
.
├── docs/                # Generated Chinese documentation output
│   ├── c/               # C language documents
│   ├── cpp/             # C++ documents
│   ├── go/              # Go documents (planned)
│   ├── java/            # Java documents (planned)
│   └── rust/            # Rust documents (planned)
├── original_docs/       # Source English documents
└── .claude/
    └── skills/
        └── cppref-synthesis/  # Core document processing skill
```

## Core Skill: cppref-synthesis

The `cppref-synthesis` skill is the primary tool for document processing. It analyzes technical documents (HTML/Markdown/Code) and generates structured Chinese documentation with 7 standard sections:

1. **概述 (Overview)** - Concept definition, primary use, technical positioning
2. **来源与演变 (Origin and Evolution)** - History, design motivation, version changes
3. **语法与参数 (Syntax and Parameters)** - Core syntax, parameter meanings
4. **底层原理 (Underlying Principles)** - Implementation, algorithms, performance
5. **使用场景 (Use Cases)** - Applicable scenarios, best practices, common pitfalls
6. **代码示例 (Examples)** - Basic usage, advanced usage, common errors with fixes
7. **总结 (Summary)** - Key points, comparisons, learning suggestions

### Using the Skill

Invoke it with: `/cppref-synthesis` or describe document analysis tasks like:
- "分析技术文档"
- "生成中文文档"
- "整理 C++ 文档"

The skill processes HTML, Markdown, and code files, outputting Chinese documentation to `docs/`.

## Document Generation Guidelines

When generating documentation:
- Use standard Markdown format with proper language tags for code blocks
- Mark English original terms on first occurrence
- All 7 sections must be present (mark as "N/A" if no content)
- Include at least one "correct usage" and one "common error" example
- Use `###` for subsection headings within chapters

## Installed Skills

From `skills-lock.json`:
- `brainstorming` - Design exploration before implementation
- `find-skills` - Discover and install new skills

## MCP Servers

Configured in `.mcp.json`:
- `github` - GitHub operations
- `context7` - Live documentation lookup
- `memory` - Persistent memory across sessions
- `filesystem` - Filesystem operations
- `sequential-thinking` - Chain-of-thought reasoning
- `supabase` - Database operations