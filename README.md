# Librarian Agents Team 

> **A multi-agent system for intelligently processing large documents**

---

## 📋 Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Detailed Usage Guide](#detailed-usage-guide)
- [Working with Large Documents](#working-with-large-documents)
- [File Structure](#file-structure)
- [Configuration](#configuration)
- [Examples](#examples)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

---

## Overview

The Librarian Agents Team is a multi-agent document processing system, intelligently breaking down, analysing, and transforming document content using specialised AI agents.

### Key Features

✅ **Multi-Agent Architecture**: Coordinated team of specialised agents  
✅ **Intelligent Task Delegation**: Automatic task breakdown and assignment  
✅ **Multiple Document Formats**: PDF, DOCX, TXT, HTML, Markdown  
✅ **Interactive & Batch Modes**: CLI interface for flexibility  
✅ **Automated Document Chunking**: Built-in `document_chunker.py` utility with smart algorithms  
✅ **Multiple Output Formats**: Markdown, HTML, CSV for tables  
✅ **Clarification Requests**: Agents can ask follow-up questions  
✅ **Context Window Management**: Handles documents up to 200K tokens automatically

### The Agent Team

1. **Lead Orchestrator Agent**
   - Coordinates all operations
   - Breaks down complex tasks
   - Delegates to specialized agents
   - Compiles final results
   - Manages context windows

2. **SubAgent 1: Text Processing Specialist**
   - Document summarisation
   - Text analysis and extraction
   - Content condensing
   - Key points identification

3. **SubAgent 2: Text Transformation Specialist**
   - Content formatting and styling
   - Document restructuring
   - Text conversion
   - Multiple format outputs

4. **SubAgent 3: Table Generation Specialist**
   - Complex table creation
   - Merged cells handling
   - Data extraction to tables
   - HTML/Markdown/CSV formats

---

## System Architecture

### Component Overview

```
┌─────────────────────────────────────────────────────────────┐
│                       USER INTERFACE                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  cli.py (Command-Line Interface)                            │
│  - Interactive mode                                         │
│  - Batch processing                                         │
│  - File upload handling                                     │
│                                                             │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ↓
┌─────────────────────────────────────────────────────────────┐
│                    UTILITY LAYER                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  document_loader.py (✅ Auto-used by CLI)                  │
│  - Loads PDF, DOCX, TXT, HTML, MD                           │
│  - Extracts text content                                    │
│  - Automatically called by CLI                              │
│                                                             │
│  document_chunker.py (⚠️ Used via wrapper scripts)          │
│  - Automated smart chunking algorithms                      │
│  - Structure detection (chapters/pages/sections)            │
│  - Must be used in multi-step workflow for >200K docs       │
│                                                             │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ↓
┌─────────────────────────────────────────────────────────────┐
│                   PROCESSING ENGINE                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  librarian_agents_team.py (Core System)                     │
│  - Lead Orchestrator Agent                                  │
│  - SubAgent 1 (Text Processing)                             │
│  - SubAgent 2 (Text Transformation)                         │
│  - SubAgent 3 (Table Generation)                            │
│  - Task delegation logic                                    │
│  - Result compilation                                       │
│                                                             │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ↓
┌─────────────────────────────────────────────────────────────┐
│                     CLAUDE API                              │
│  - Model: Claude Haiku 4.5 or Sonnet 4.5                    │
│  - 200K token context window                                │
│  - 64K token max output                                     │
│  - Prompt caching support                                   │
└─────────────────────────────────────────────────────────────┘
```

### Workflow Diagram

```
User uploads document (PDF/DOCX/TXT)
              ↓
    [document_loader.py loads automatically] ✅
              ↓
    Is document > 200K tokens?
              ↓
         ┌────┴────┐
         │         │
        NO        YES
         │         │
         │    [MULTI-STEP WORKFLOW REQUIRED] ⚠️
         │    - Use document_chunker.py utility
         │    - Process chunks separately
         │    - Combine results
         │         │
         └────┬────┘
              ↓
    [librarian_agents_team.py processes]
              ↓
    Lead Orchestrator analyzes request
              ↓
    Creates task breakdown
              ↓
    Delegates to SubAgents
              ↓
    SubAgents process assigned tasks
              ↓
    Lead compiles results
              ↓
    Returns final output
              ↓
    User saves results
```

---

## Prerequisites

### System Requirements

- **Python**: 3.8 or higher
- **Operating System**: Windows, macOS, or Linux

### Required Accounts

- **Anthropic API Account**: Required for Claude AI access
- **API Key**: You'll need a valid `ANTHROPIC_API_KEY`

### Python Dependencies

- `anthropic` >= 0.40.0

---

## Installation

### Step 1: Clone or Download the Repository

```bash
git clone https://github.com/wei-kiat-tan/Agentic-Librarian.git
cd Agentic-Librarian
```

Or download and extract the ZIP file.

### Step 2: Install Dependencies

#### On Linux/Mac:
```bash
pip3 install anthropic --break-system-packages
```

#### On Windows:
```bash
pip install anthropic --break-system-packages
```

Or if you prefer using Python's package manager directly:
```bash
python -m pip install anthropic
```

### Step 3: Set Up API Key

#### Method 1: Environment Variable (Recommended)

**Linux/Mac:**
```bash
export ANTHROPIC_API_KEY='your-api-key-here'

# Add to ~/.bashrc or ~/.zshrc for persistence
echo "export ANTHROPIC_API_KEY='your-api-key-here'" >> ~/.bashrc
source ~/.bashrc
```

**Windows (PowerShell):**
```powershell
$env:ANTHROPIC_API_KEY='your-api-key-here'

# Or permanently set it:
setx ANTHROPIC_API_KEY 'your-api-key-here'
```

**Windows (Command Prompt):**
```cmd
set ANTHROPIC_API_KEY=your-api-key-here

# Or permanently:
setx ANTHROPIC_API_KEY "your-api-key-here"
```

**Windows (System Environment Variables - GUI method):**
1. Search for "Edit the system environment variables"
2. Click "Environment Variables"
3. Under "User variables", click "New"
4. Variable name: `ANTHROPIC_API_KEY`
5. Variable value: `your-api-key-here`
6. Click OK

#### Method 2: In Your Python Script
```python
import os
os.environ['ANTHROPIC_API_KEY'] = 'your-api-key-here'
```

⚠️ **IMPORTANT**: After setting environment variables, **restart your terminal** for changes to take effect.

### Step 4: Verify Installation

```bash
# Check if Python is installed
python --version
# or
python3 --version

# Check if API key is set
echo $ANTHROPIC_API_KEY  # Linux/Mac
echo %ANTHROPIC_API_KEY%  # Windows CMD
$env:ANTHROPIC_API_KEY    # Windows PowerShell
```

---

## Quick Start

### Basic Usage (Interactive Mode)

```bash
python cli.py -i document.pdf --interactive
```

**What happens:**
1. CLI loads your PDF automatically (via `document_loader.py`)
2. Extracts text content
3. Prompts you for requests
4. Processes with AI agents
5. Returns results

### Example Session

```bash
$ python cli.py -i "your-document.pdf" --interactive

📄 Loading document...
✓ Loaded 25,000 characters (~6,250 tokens)
🤖 Initializing Librarian Agents Team...

============================================================
Interactive Mode - Librarian Agents Team
============================================================
Document loaded successfully!
Document: your-document.pdf
Size: 25000 characters

Type your request or 'quit' to exit
============================================================

Your request: Summarize the main arguments in 3 paragraphs

🤖 Processing request...

[SYSTEM] Lead Orchestrator analyzing request...
[SYSTEM] Created 1 tasks
[SYSTEM] Delegating to subagents...
[SYSTEM] SubAgent 1 processing: Summarize main arguments
[SYSTEM] Lead Orchestrator compiling final output...

============================================================
RESULT
============================================================

[Summary appears here]

============================================================

Save this result? (y/N): y
Output file path: summary.txt
✓ Saved to summary.txt

Your request: quit

Goodbye! 👋
```

---

## Detailed Usage Guide

### Command-Line Interface Options

```bash
python cli.py [OPTIONS]
```

#### Options:

| Option | Description | Example |
|--------|-------------|---------|
| `-i, --input` | Input document path (required) | `-i document.pdf` |
| `-r, --request` | Processing request (batch mode) | `-r "Summarize this"` |
| `-o, --output` | Output file path (batch mode) | `-o summary.txt` |
| `--interactive` | Interactive mode | `--interactive` |
| `--verbose` | Show detailed processing logs | `--verbose` |
| `--metadata` | Show document metadata | `--metadata` |

### Usage Modes

#### Mode 1: Interactive (Recommended)

```bash
python cli.py -i document.pdf --interactive
```

**Features:**
- Multiple requests on same document
- Answer clarification questions
- Choose when to save results
- Exit anytime with `quit`

**Best for:**
- Exploration and experimentation
- Complex multi-step tasks
- When you need to refine requests

---

#### Mode 2: Batch Processing

```bash
python cli.py -i document.pdf -r "Summarize this document" -o output.txt
```

**Features:**
- Single command execution
- No interaction required
- Automatic file saving
- Good for scripting

**Best for:**
- Automated workflows
- Scripting and cron jobs
- Simple, one-off tasks

---

#### Mode 3: With Verbose Output

```bash
python cli.py -i document.pdf --interactive --verbose
```

**Shows:**
- Agent activity details
- Task delegation information
- Processing steps
- Token usage (if available)

**Best for:**
- Debugging
- Understanding the process
- Development and testing

---

### Handling Different File Types

#### PDF Documents

```bash
python cli.py -i report.pdf --interactive
```

✅ **Automatically extracts text** via `document_loader.py`  
✅ **No additional steps needed**

---

#### Word Documents (.docx)

```bash
python cli.py -i thesis.docx --interactive
```

✅ **Automatically reads content** including text and tables

---

#### Text Files

```bash
python cli.py -i notes.txt --interactive
```

✅ **Direct reading**, fastest loading time

---

#### HTML Files

```bash
python cli.py -i webpage.html --interactive
```

✅ **Extracts text content** from HTML tags

---

#### Markdown Files

```bash
python cli.py -i readme.md --interactive
```

✅ **Preserves structure** including headers and lists

---

### Responding to Clarification Questions

When agents need more information:

```
Your request: Create a comparison table

[Agent processes]

============================================================
RESULT
============================================================

**SubAgent 3** needs clarification for:
Task: Create comparison table
Question: What columns should the table include? What items to compare?

============================================================

Save this result? (y/N): N  ← Type N (don't save the question)

Your request: Insert your clarification or prompt here

[Agent processes with your clarification]

[Final table generated]

Save this result? (y/N): y
Output file path: phone_comparison.html
✓ Saved
```

**Key Points:**
- Type `N` when asked to save clarification questions
- Provide detailed answers to clarification prompts
- Be specific to get best results

---

### Saving Results

#### Choosing File Formats

**For plain text summaries:**
```
Output file path: summary.txt
```

**For tables (recommended HTML):**
```
Output file path: comparison_table.html
```
Then open in browser for beautiful formatting!

**For structured content (Markdown):**
```
Output file path: analysis.md
```
Open in VS Code or Markdown viewer

#### File Path Guidelines

**Save in current directory:**
```
Output file path: result.txt
```

**Save to Desktop (Windows):**
```
Output file path: C:\Users\YourName\Desktop\result.html
```

**Save to Desktop (Mac):**
```
Output file path: /Users/YourName/Desktop/result.html
```

**Save to specific folder:**
```
Output file path: C:\Users\YourName\Documents\Projects\output.txt
```

⚠️ **Important:** Do NOT include quotes around file paths when saving interactively!


---

### 🔍 Known Issues

#### Issue 1: Terminal Output Formatting

**Problem:** Tables and formatted content are hard to read in terminal

**Workaround:**
- Save output as HTML: `Output file path: result.html`
- Open HTML file in browser for proper formatting
- Or request plain text format in your prompt

---

#### Issue 2: Windows Path Issues

**Problem:** File paths with spaces or special characters may cause issues

**Solution:**
- Use forward slashes: `C:/Users/Your Name/file.pdf`
- Or escape backslashes: `C:\\Users\\Your Name\\file.pdf`
- Do NOT include quotes in file paths when saving interactively

---

#### Issue 3: No Progress Indicator for Long Processing

**Problem:** No visual feedback during long-running tasks

**Workaround:**
- Use `--verbose` flag to see processing steps
- Be patient - large documents take 30 seconds to several minutes

---

## Working with Large Documents

### Understanding Token Limits

**Rule of thumb:**
- **1 token ≈ 4 characters**
- **1 page ≈ 2,000 characters ≈ 500 tokens**
- **200K tokens ≈ 800,000 characters ≈ 150-200 pages**

**Quick size check:**

```python
# Quick token estimator
with open('document.txt', 'r', encoding='utf-8') as f:
    content = f.read()
    estimated_tokens = len(content) / 4
    print(f"Estimated tokens: {estimated_tokens:,.0f}")
    
    if estimated_tokens > 200000:
        print("⚠️ Document too large - multi-step workflow required")
    elif estimated_tokens > 180000:
        print("⚠️ Document large - may need multi-step workflow")
    else:
        print("✅ Document size OK")
```

---

### The `document_chunker.py` Utility

Your repository includes a powerful document chunking utility with **automated algorithms**:

#### **Automated Chunking Methods:**

1. **`smart_chunk(content)`** - Automatically detects document structure and splits optimally
   - Auto-detects chapters, pages, or sections
   - Preserves document structure
   - Finds logical split points

2. **`chunk_by_chapters(content)`** - Splits on chapter boundaries
   - Detects "CHAPTER 1", "Chapter I", etc.
   - Preserves chapter integrity

3. **`chunk_by_pages(content)`** - Splits by page markers
   - Works with documents containing "--- Page X ---" markers

4. **`chunk_by_sections(content)`** - Splits by paragraphs
   - Splits at paragraph boundaries
   - Good for unstructured documents

#### **What's Automated:**
- ✅ Structure detection (chapters/pages/sections)
- ✅ Intelligent split point identification
- ✅ Chunk size optimization
- ✅ Metadata preservation

#### **What Requires Your Action:**
- ❌ Creating wrapper scripts to use the utility (one-time setup)
- ❌ Running the chunking workflow
- ❌ Processing each chunk with CLI (not integrated)
- ❌ Combining results

---

### Multi-Step Workflow for Large Documents (>200K tokens)

⚠️ **Required for documents over 200K tokens (approx. 150-200 pages)**

The `document_chunker.py` utility provides automated chunking algorithms, but you need to orchestrate the workflow yourself.

---

#### **One-Time Setup: Create Wrapper Scripts**

You only need to create these scripts ONCE, then reuse them for all large documents.

---

#### **Script 1: `chunk_large_doc.py` (20 lines)**

This wrapper uses the automated chunking algorithms from `document_chunker.py`:

```python
#!/usr/bin/env python3
"""
Wrapper script to chunk large documents using document_chunker.py utility.
Uses automated smart chunking algorithms to split documents intelligently.
"""

import sys
from document_chunker import DocumentChunker, load_document, save_document

if len(sys.argv) < 2:
    print("Usage: python chunk_large_doc.py <document_path>")
    print("Example: python chunk_large_doc.py huge_book.pdf")
    sys.exit(1)

# Load document
file_path = sys.argv[1]
print(f"📄 Loading document: {file_path}")
content = load_document(file_path)

print(f"   Size: {len(content):,} characters")
print(f"   Estimated tokens: {len(content) / 4:,.0f}")

# Check if chunking needed
estimated_tokens = len(content) / 4
if estimated_tokens < 180000:
    print("\n✅ Document size OK - no chunking needed!")
    print(f"   Process normally with: python cli.py -i {file_path} --interactive")
    sys.exit(0)

print("\n⚠️  Document exceeds 180K token threshold")
print("   Multi-step workflow required\n")

# Use automated smart_chunk algorithm
print("🔄 Running automated chunking algorithm...")
chunker = DocumentChunker(max_chunk_size=600000)  # ~150K tokens per chunk
chunks = chunker.smart_chunk(content)  # Automated structure detection!

print(f"✅ Automated chunking complete!")
print(f"   Split into {len(chunks)} chunks\n")

# Save each chunk
for i, chunk in enumerate(chunks):
    filename = f'chunk_{i}.txt'
    save_document(chunk['content'], filename)
    tokens = len(chunk['content']) / 4
    print(f"   ✓ Saved {filename} ({len(chunk['content']):,} chars, ~{tokens:,.0f} tokens)")

print("\n" + "="*70)
print("NEXT STEPS:")
print("="*70)
print("Process each chunk with CLI:\n")
for i in range(len(chunks)):
    print(f"  python cli.py -i chunk_{i}.txt --interactive")
    print(f"  [Save result as result_{i}.txt]\n")
print("Then combine results with:")
print("  python combine_results.py")
print("="*70)
```

**Save this as `chunk_large_doc.py` in your project directory.**

---

#### **Script 2: `combine_results.py` (15 lines)**

This wrapper uses the `ChunkMerger` class from `document_chunker.py`:

```python
#!/usr/bin/env python3
"""
Combine processed chunk results using ChunkMerger from document_chunker.py.
Automatically adds section headers between chunks.
"""

from document_chunker import ChunkMerger

# Auto-detect all result files
results = []
i = 0
while True:
    try:
        with open(f'result_{i}.txt', 'r', encoding='utf-8') as f:
            results.append({'content': f.read(), 'chunk_id': i})
        i += 1
    except FileNotFoundError:
        break

if not results:
    print("❌ No result files found!")
    print("   Expected: result_0.txt, result_1.txt, etc.")
    exit(1)

print(f"📄 Found {len(results)} result files")

# Use ChunkMerger to combine with headers
merger = ChunkMerger()
combined = merger.merge_with_headers(results)

# Save combined result
output_file = 'final_combined_summary.txt'
with open(output_file, 'w', encoding='utf-8') as f:
    f.write(combined)

print(f"✅ Combined {len(results)} results")
print(f"✅ Saved to: {output_file}")
```

**Save this as `combine_results.py` in your project directory.**

---

#### **Using the Multi-Step Workflow**

Once you've created the two wrapper scripts above (one-time setup), here's how to process large documents:

---

#### **Step 1: Chunk Your Large Document**

```bash
python chunk_large_doc.py huge_document.pdf
```

**Example output:**
```
📄 Loading document: huge_document.pdf
   Size: 1,600,000 characters
   Estimated tokens: 400,000

⚠️  Document exceeds 180K token threshold
   Multi-step workflow required

🔄 Running automated chunking algorithm...
✅ Automated chunking complete!
   Split into 3 chunks

   ✓ Saved chunk_0.txt (600,000 chars, ~150,000 tokens)
   ✓ Saved chunk_1.txt (600,000 chars, ~150,000 tokens)
   ✓ Saved chunk_2.txt (400,000 chars, ~100,000 tokens)

======================================================================
NEXT STEPS:
======================================================================
Process each chunk with CLI:

  python cli.py -i chunk_0.txt --interactive
  [Save result as result_0.txt]

  python cli.py -i chunk_1.txt --interactive
  [Save result as result_1.txt]

  python cli.py -i chunk_2.txt --interactive
  [Save result as result_2.txt]

Then combine results with:
  python combine_results.py
======================================================================
```

**What happened:**
- ✅ `load_document()` loaded your PDF automatically
- ✅ `smart_chunk()` automatically detected structure and split intelligently
- ✅ Chunks saved as separate files
- ⚠️ Now you need to process each chunk

---

#### **Step 2: Process Each Chunk with CLI**

**Process Chunk 0:**
```bash
python cli.py -i chunk_0.txt --interactive

Your request: Summarize the content providing key points in a concise manner

🤖 Processing request...

[Summary of chunk 0 generated]

Save this result? y
Output file path: result_0.txt
✓ Saved to result_0.txt

Your request: quit
```

**Process Chunk 1:**
```bash
python cli.py -i chunk_1.txt --interactive

Your request: Summarize the content providing key points in a concise manner

[Summary of chunk 1 generated]

Save this result? y
Output file path: result_1.txt
✓ Saved to result_1.txt

Your request: quit
```

**Process Chunk 2:**
```bash
python cli.py -i chunk_2.txt --interactive

Your request: Summarize the content providing key points in a concise manner

[Summary of chunk 2 generated]

Save this result? y
Output file path: result_2.txt
✓ Saved to result_2.txt

Your request: quit
```

**Important:** Use the SAME request for each chunk to ensure consistent processing!

---

#### **Step 3: Combine Results**

```bash
python combine_results.py
```

**Output:**
```
📄 Found 3 result files
✅ Combined 3 results
✅ Saved to: final_combined_summary.txt
```

**The combined file will have section headers:**
```
=== Section 1 ===
[Summary from chunk 0]

=== Section 2 ===
[Summary from chunk 1]

=== Section 3 ===
[Summary from chunk 2]
```

---

### Workflow Summary

```
┌──────────────────────────────────────────────────────────┐
│ ONE-TIME SETUP (Create wrapper scripts)                  │
├──────────────────────────────────────────────────────────┤
│ 1. Create chunk_large_doc.py (20 lines)                  │
│ 2. Create combine_results.py (15 lines)                  │
│                                                          │
│ ✅ Do this once, reuse forever                          │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│ FOR EACH LARGE DOCUMENT                                  │
├──────────────────────────────────────────────────────────┤
│ Step 1: python chunk_large_doc.py file.pdf               │
│         └─> Uses automated chunking algorithms ✅       │
│                                                          │
│ Step 2: python cli.py -i chunk_0.txt --interactive       │
│         [Save as result_0.txt]                           │
│                                                          │
│ Step 3: python cli.py -i chunk_1.txt --interactive       │
│         [Save as result_1.txt]                           │
│                                                          │
│ Step 4: python cli.py -i chunk_2.txt --interactive       │
│         [Save as result_2.txt]                           │
│                                                          │
│ Step 5: python combine_results.py                        │
│         └─> Automated merging with headers ✅           │
└──────────────────────────────────────────────────────────┘
```

---

### Key Points

✅ **What's Automated:**
- Document loading
- Smart structure detection
- Intelligent chunk splitting
- Result merging with headers

❌ **What Requires Manual Steps:**
- Recognizing document is too large
- Running chunking wrapper
- Processing each chunk separately (not integrated into CLI)
- Running combining wrapper

**The chunking algorithms are automated, but the workflow is not integrated into the CLI.**

---

### Alternative: Direct Python Script

If you prefer a single script approach instead of CLI interactive mode:

```python
# process_large_doc.py
from librarian_agents_team import LibrarianAgentsTeam
from document_chunker import DocumentChunker, load_document

# Load document
content = load_document("huge_document.pdf")
estimated_tokens = len(content) / 4

print(f"Document: {estimated_tokens:,.0f} tokens")

if estimated_tokens > 180000:
    print("Chunking required...")
    
    # Use automated chunking
    chunker = DocumentChunker(max_chunk_size=600000)
    chunks = chunker.smart_chunk(content)
    print(f"Split into {len(chunks)} chunks")
    
    # Process each chunk
    team = LibrarianAgentsTeam()
    request = "Summarize providing key points"
    
    results = []
    for i, chunk in enumerate(chunks):
        print(f"Processing chunk {i+1}/{len(chunks)}...")
        result = team.process_document(request, chunk['content'])
        results.append(result)
    
    # Combine results
    combined = "\n\n=== CHUNK SEPARATOR ===\n\n".join(results)
    
    with open('combined_summary.txt', 'w') as f:
        f.write(combined)
    
    print("✅ Complete! Saved to combined_summary.txt")
else:
    print("Size OK - processing normally...")
    team = LibrarianAgentsTeam()
    result = team.process_document("Summarize", content)
    print(result)
```

---

## File Structure

```
Agentic-Librarian/
│
├── README.md                      # This file - main documentation
├── QUICKSTART.md                  # Quick setup guide
├── ARCHITECTURE.md                # System architecture details
├── EXECUTION_FLOW.md              # Execution flow diagrams
├── PROJECT_SUMMARY.md             # Project overview
├── USAGE_GUIDE.md                 # Detailed usage instructions
│
├── librarian_agents_team.py       # 🤖 CORE: Agent system
│   └── Contains:
│       ├── LeadOrchestratorAgent
│       ├── SubAgent1 (Text processing)
│       ├── SubAgent2 (Text transformation)
│       ├── SubAgent3 (Table generation)
│       └── LibrarianAgentsTeam (main orchestrator)
│
├── cli.py                         # 💻 CLI: Command-line interface
│   └── User-facing entry point
│   └── Automatically uses document_loader.py
│
├── document_loader.py             # 📂 UTILITY: File loading (auto-used)
│   └── Loads PDF, DOCX, TXT, HTML, MD
│   └── Automatically called by cli.py
│   └── You never run this directly
│
├── document_chunker.py            # ✂️ UTILITY: Automated chunking
│   └── DocumentChunker class with smart algorithms
│   └── ChunkMerger class for combining results
│   └── Used via wrapper scripts for >200K docs
│   └── Provides automated chunking, not integrated into CLI
│
├── advanced_examples.py           # 📚 EXAMPLES: Complex patterns
├── test_example.py                # 🧪 TESTING: Example tests
│
├── [Wrapper scripts - you create these]
│   ├── chunk_large_doc.py         # Uses document_chunker.py
│   └── combine_results.py         # Uses ChunkMerger
│
└── [Generated files during workflow]
    ├── chunk_0.txt                # Created by chunking workflow
    ├── chunk_1.txt
    ├── result_0.txt               # Processing results
    ├── result_1.txt
    └── final_combined_summary.txt # Combined output
```

### File Purposes

| File | Purpose | Run Directly? | Used Automatically? |
|------|---------|---------------|-------------------|
| `cli.py` | User interface | ✅ YES | N/A |
| `librarian_agents_team.py` | Agent engine | ❌ NO | ✅ By CLI |
| `document_loader.py` | Load files | ❌ NO | ✅ By CLI |
| `document_chunker.py` | Automated chunking utility | ❌ NO | ⚠️ Via wrappers |
| `chunk_large_doc.py` | Wrapper for chunking | ✅ YES | ❌ You run it |
| `combine_results.py` | Wrapper for merging | ✅ YES | ❌ You run it |
| `test_example.py` | Testing | ✅ YES | N/A |
| `advanced_examples.py` | Learning | ✅ YES | N/A |

---

## Configuration

### Model Selection

Edit `librarian_agents_team.py` (line ~17):

```python
# Current model
MODEL = "claude-haiku-4-5-20251001"  # Haiku 4.5: $1/$5 per M tokens

# Alternative: Sonnet 4.5 (more capable, 3x cost)
# MODEL = "claude-sonnet-4-5-20250929"  # Sonnet 4.5: $3/$15 per M tokens
```

**Model Comparison:**

| Model | Input Cost | Output Cost | Best For |
|-------|-----------|-------------|----------|
| **Haiku 4.5** | $1/M | $5/M | Cost efficiency, PoC, high volume |
| **Sonnet 4.5** | $3/M | $15/M | Complex reasoning, quality critical |

---

### Adjusting Output Token Limits

Edit `librarian_agents_team.py`:

```python
# In each agent's process() method, adjust max_tokens:

# Current settings (Haiku 4.5 optimized):
max_tokens=32000

# For longer outputs, increase to:
max_tokens=64000  # Maximum for Haiku 4.5 / Sonnet 4.5
```

**Recommendations:**

| Agent | Current | Recommended Range | Notes |
|-------|---------|------------------|-------|
| Lead Analyzer | 16,000 | 8,000 - 16,000 | Task planning |
| Lead Compiler | 32,000 | 16,000 - 64,000 | Final output |
| SubAgents | 32,000 | 16,000 - 64,000 | Content generation |

---

### Chunking Configuration

Edit `chunk_large_doc.py` to adjust chunk sizes:

```python
# Conservative (smaller chunks, more chunks)
chunker = DocumentChunker(max_chunk_size=400000)  # ~100K tokens

# Balanced (recommended)
chunker = DocumentChunker(max_chunk_size=600000)  # ~150K tokens

# Aggressive (larger chunks, fewer chunks)
chunker = DocumentChunker(max_chunk_size=720000)  # ~180K tokens
```

---

## Examples

### Example 1: Simple Summarization

```bash
python cli.py -i research_paper.pdf --interactive

Your request: Create a 500-word executive summary highlighting the main findings

[Processing...]
[Summary generated]

Save this result? y
Output file path: executive_summary.txt
```

---

### Example 2: Extract Data to Table

```bash
python cli.py -i sales_report.pdf --interactive

Your request: Extract all quarterly sales data and create an HTML table with columns: Quarter, Revenue, Growth%, Top Product. Use proper table formatting with merged headers.

[Processing...]
[HTML table generated]

Save this result? y
Output file path: sales_table.html
```

Open `sales_table.html` in browser for beautifully formatted table!

---

### Example 3: Content Restructuring

```bash
python cli.py -i meeting_notes.txt --interactive

Your request: Restructure these notes into:
1. Executive Summary (3 sentences)
2. Action Items (bulleted list with owners)
3. Decisions Made (numbered list)
4. Next Steps (with timeline)

Format as Markdown.

[Processing...]

Save this result? y
Output file path: structured_notes.md
```

---

### Example 4: Comparative Analysis

```bash
python cli.py -i product_specs.pdf --interactive

Your request: Create a comparison table analyzing Product A, Product B, and Product C across these dimensions: Price, Features, Performance, User Rating, Pros, Cons. Use HTML format with color coding for ratings.

[Processing with possible clarification]

SubAgent 3 needs clarification:
What color scheme should be used for ratings?

Your request: Use green for ratings 4-5, yellow for 3-3.9, red for below 3

[Final table generated]

Save this result? y
Output file path: product_comparison.html
```

---

### Example 5: Large Document (Multi-Step Workflow Required)

```bash
# Step 1: Check document size
python -c "
with open('huge_book.txt', 'r') as f:
    content = f.read()
    tokens = len(content) / 4
    print(f'Estimated tokens: {tokens:,.0f}')
    if tokens > 200000:
        print('⚠️ Multi-step workflow required!')
"

# Output: Estimated tokens: 450,000
#         ⚠️ Multi-step workflow required!

# Step 2: Use automated chunking
python chunk_large_doc.py huge_book.txt

# Output: Split into 3 chunks
#         chunk_0.txt, chunk_1.txt, chunk_2.txt created

# Step 3: Process each chunk
python cli.py -i chunk_0.txt --interactive
Your request: Summarize providing key points
[Save as result_0.txt]

python cli.py -i chunk_1.txt --interactive
Your request: Summarize providing key points
[Save as result_1.txt]

python cli.py -i chunk_2.txt --interactive
Your request: Summarize providing key points
[Save as result_2.txt]

# Step 4: Combine results with automated merging
python combine_results.py

# Output: ✅ Combined 3 results
#         ✅ Saved to: final_combined_summary.txt
```

---

## Troubleshooting

### Issue: "Python not found" Error (Windows)

**Solution 1:** Use `py` instead:
```bash
py cli.py -i document.pdf --interactive
```

**Solution 2:** Add Python to PATH:
1. Find Python installation location
2. Add to system PATH environment variable
3. Restart terminal

---

### Issue: "ANTHROPIC_API_KEY not set"

**Error:**
```
Error: ANTHROPIC_API_KEY environment variable not set
```

**Solution:**
```bash
# Check if set
echo $ANTHROPIC_API_KEY  # Linux/Mac
echo %ANTHROPIC_API_KEY%  # Windows

# If empty, set it
export ANTHROPIC_API_KEY='your-key'  # Linux/Mac
setx ANTHROPIC_API_KEY "your-key"    # Windows

# Restart terminal
```

---

### Issue: Document Loading Fails

**Error:**
```
Error loading document: [file_path]
```

**Solutions:**

1. **Check file path:**
   ```bash
   # Use absolute path
   python cli.py -i "C:\Users\YourName\Documents\file.pdf" --interactive
   
   # Or relative path
   python cli.py -i ./documents/file.pdf --interactive
   ```

2. **Check file exists:**
   ```bash
   # Windows
   dir "path\to\file.pdf"
   
   # Linux/Mac
   ls "path/to/file.pdf"
   ```

3. **Check file permissions:**
   - Ensure file is not locked by another program
   - Ensure you have read permissions

---

### Issue: "exceeds 200000 token limit" Error

**Error:**
```
anthropic.BadRequestError: messages: total length of messages must be at most 200000 tokens. Your request was 400000 tokens.
```

**This means:** Your document is too large for automatic processing.

**Solution:** Use multi-step workflow with `document_chunker.py` (see "Working with Large Documents" section)

---

### Issue: Output Cut Off

**Problem:** Agent response appears incomplete

**Solutions:**

1. **Reply with "continue":**
   ```
   Your request: continue
   ```

2. **Use smaller chunks:**
   - Process document in smaller sections
   - Request shorter summaries

3. **Increase max_tokens:**
   - Edit `librarian_agents_team.py`
   - Increase `max_tokens` parameter

---

### Issue: Tables Not Formatted Properly in Terminal

**Problem:** Tables look messy in terminal output

**Solutions:**

1. **Save as HTML (recommended):**
   ```
   Output file path: table.html
   ```
   Then open in browser

2. **Request simple format:**
   ```
   Your request: Create a simple Markdown table (no merged cells)
   ```

3. **Use CSV for data:**
   ```
   Your request: Export data as CSV format
   ```

---

### Issue: Agent Asks for Clarification Repeatedly

**Problem:** Agent keeps asking follow-up questions

**Solution:** Be more specific in initial request:

**Instead of:**
```
Your request: Create a table
```

**Use:**
```
Your request: Create an HTML table with columns: Name, Age, Location, Email. Include data for all employees mentioned in the document. Use blue header background with white text.
```

---

### Issue: Slow Processing

**Problem:** Processing takes a long time

**Expected processing times:**
- Small documents (< 10 pages): 10-30 seconds
- Medium documents (10-50 pages): 30 seconds - 2 minutes
- Large documents (50-150 pages): 2-5 minutes
- Very large documents (chunked workflow): 5-15 minutes

**If slower than expected:**
- Check internet connection
- Verify API key is valid
- Use `--verbose` to see where it's slow
- Consider using smaller chunks

---

## Best Practices

### 1. Document Preparation

✅ **DO:**
- Use clear, readable text documents
- Ensure PDFs have selectable text (not scanned images)
- Check document size before processing (use token estimator)
- Remove unnecessary content

❌ **DON'T:**
- Use scanned PDFs without OCR
- Include large amounts of irrelevant content
- Upload documents with DRM protection

---

### 2. Writing Effective Requests

✅ **DO:**
```
Your request: Create a summary with these specifications:
- Length: 500 words maximum
- Focus: Key findings and conclusions
- Audience: Executive leadership
- Format: 3 paragraphs with bullet points for key takeaways
- Tone: Professional and concise
```

❌ **DON'T:**
```
Your request: Summarize this
```

---

### 3. Working with Tables

✅ **DO:**
- Specify exact column names
- Mention format (HTML/Markdown/CSV)
- Indicate if merged cells needed
- Describe any special formatting

**Example:**
```
Your request: Create an HTML table with:
- Columns: Product Name, Q1 Sales, Q2 Sales, Q3 Sales, Q4 Sales, Total, Growth%
- Merged header row with "Quarterly Sales" spanning Q1-Q4
- Color-code Growth%: green if >10%, yellow if 5-10%, red if <5%
- Include summary row with totals
```

---

### 4. Handling Large Documents

✅ **DO:**
- Check document size first with token estimator
- Use multi-step workflow for documents > 180K tokens
- Use automated chunking via `document_chunker.py`
- Keep chunk filenames organized (chunk_0, chunk_1, etc.)
- Use same request for all chunks for consistency

❌ **DON'T:**
- Try to process 400K+ token documents without chunking
- Mix up chunk order
- Use different requests for different chunks (causes inconsistency)
- Forget to combine results at the end

---

### 5. Saving Results

✅ **DO:**
- Use descriptive filenames: `sales_analysis_Q4_2024.html`
- Choose appropriate formats (HTML for tables, TXT for text)
- Save to organized folders
- Keep original requests documented

❌ **DON'T:**
- Use generic names: `output.txt`
- Mix file formats inappropriately
- Forget to save important results

---

### 6. Cost Management

For Haiku 4.5 ($1 input / $5 output per million tokens):

**Typical costs:**
- Small document (10 pages, simple summary): $0.01 - $0.05
- Medium document (50 pages, detailed analysis): $0.05 - $0.20
- Large document (150 pages, multiple tasks): $0.20 - $0.50
- Very large document (400+ pages, chunked): $0.50 - $2.00

**Tips to reduce costs:**
- Use Haiku 4.5 instead of Sonnet 4.5 (3x cheaper)
- Process only necessary sections
- Combine related requests
- Use batch mode for simple tasks
- Chunk efficiently (fewer, larger chunks = fewer API calls)

---

#### 3. Progress Indicators

**Current:** Silent processing  
**Planned:** Visual progress feedback

**Expected behavior:**
```
Processing chunk 2/5... [████████░░] 40% complete
Estimated time remaining: 2 minutes
```

---

#### 4. Improved Error Messages

**Current:** Technical API errors  
**Planned:** User-friendly error messages with solutions

---

## Additional Resources

### Documentation Files

- **[QUICKSTART.md](QUICKSTART.md)** - Get started in 3 minutes
- **[ARCHITECTURE.md](ARCHITECTURE.md)** - System architecture deep-dive
- **[EXECUTION_FLOW.md](EXECUTION_FLOW.MD)** - Detailed execution sequences
- **[USAGE_GUIDE.md](USAGE_GUIDE.md)** - Visual usage guide with examples

### Getting Help

**For issues or questions:**
1. Check the [Troubleshooting](#troubleshooting) section
2. Review [Examples](#examples) for similar use cases
3. Check GitHub Issues (if public)
4. Review Anthropic's API documentation

---

## Summary

### ✅ What Works Automatically

1. **PDF/DOCX/TXT Loading** - Just specify file path (via `document_loader.py`)
2. **Multi-agent Processing** - Automatic task delegation
3. **Interactive Mode** - Ask multiple questions on same document
4. **Clarification Handling** - Agents ask follow-up questions
5. **Result Compilation** - Automatic combining of agent outputs
6. **Multiple Formats** - Automatic handling of different file types

### ✅ What's Provided (Ready to Use)

1. **`document_chunker.py`** - Automated chunking utility with smart algorithms
2. **`ChunkMerger`** - Automated result merging with headers
3. **Example wrapper scripts** - Simple 20-line and 15-line scripts to create

### ⚠️ What Requires Multi-Step Workflow

**For documents >200K tokens (~150 pages):**
1. Create wrapper scripts (one-time, ~35 lines total)
2. Run chunking wrapper (uses automated `smart_chunk()`)
3. Process each chunk with CLI (not integrated)
4. Run combining wrapper (uses automated `ChunkMerger`)

**Total: 5 steps to orchestrate, but chunking itself is automated**

### 🎯 Recommended Workflow

**For documents < 150 pages:**
```bash
python cli.py -i document.pdf --interactive
# Just works! ✅
```

**For documents > 150 pages:**
```bash
# One-time: Create chunk_large_doc.py and combine_results.py

# For each large document:
python chunk_large_doc.py huge.pdf        # Automated chunking ✅
python cli.py -i chunk_0.txt --interactive
python cli.py -i chunk_1.txt --interactive
python cli.py -i chunk_2.txt --interactive
python combine_results.py                 # Automated merging ✅
```

---

**Built with Claude AI for intelligent document processing** 🚀

**Version:** 1.4  
**Last Updated:** November 24, 2024  
**Model Support:** Claude Haiku 4.5 / Sonnet 4.5
