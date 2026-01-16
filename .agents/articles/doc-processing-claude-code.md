# Claude Code's Research Capabilities on File System Documents

## Overview
This document explains how Claude Code researches and analyzes documents in file systems, using the 3GPP documents in `/data/files` as a real-world example.

---

## Architecture: Multi-Layer Research System

```
User Question
    ↓
Claude's Research Strategy
    ↓
Tool Selection & Execution
    ↓
Results Synthesis
    ↓
Answer to User
```

---

## Research Workflow Phases

### **Phase 1: Discovery - "What exists?"**

**Objective**: Map the landscape of available documents

**Tools Used**:
- `Bash(ls)` - List files
- `Glob` - Pattern-based file finding
- `Bash(find)` - Complex file searches

**Example from your folder**:
```bash
# Discovery commands I would run:
ls -la                           # See all files with metadata
ls *.docx | wc -l               # Count documents (45+ .docx files)
ls *.xlsx                       # Find spreadsheets (evaluation data)
ls *.md                         # Find analysis files
```

**Output**: Inventory of resources
- 45+ R1-*.docx files (technical contributions)
- 9 evaluation spreadsheets (.xlsx)
- 3 analysis files (.md)
- 1 PDF (JHUAPL)

---

### **Phase 2: Metadata Extraction - "Who created what?"**

**Objective**: Extract document metadata without reading full contents

**Tools Used**:
- `Bash(pandoc + head)` - Convert .docx, read first N lines
- `Grep` with context - Extract specific fields

**Strategy**:
```bash
# Extract "Source:" field from first 25 lines of each document
for f in R1-*.docx; do
  pandoc "$f" -t plain | head -25 | grep -i "^Source:"
done
```

**Why This Works**:
- 3GPP documents follow standardized format
- Source/Author appears in first 20-30 lines
- Avoids reading entire document (efficiency)
- pandoc converts binary .docx → searchable text

**Example Result**:
```
Source: Qualcomm Incorporated
Source: Viasat Satellite Holdings Ltd, Inmarsat
Source: Thales
Source: Ericsson
...
```

**Output**: Company-to-document mapping

---

### **Phase 3: Content Search - "Where is specific information?"**

**Objective**: Locate documents containing specific topics/keywords

**Tool Selection Logic**:

| Search Type | Tool | Reason |
|-------------|------|--------|
| Simple keyword | `Grep` | Fast, regex support |
| Multiple patterns | `Grep` with alternation | `"Viasat\|ViaSat\|VIASAT"` |
| Binary files (.docx) | `Bash(pandoc + grep)` | Must convert first |
| Structured extraction | `Bash(pandoc + grep -A)` | Get context around matches |

**Real Example from Session**:
```bash
# Search for "Viasat" in all .docx files
for f in R1-*.docx; do
  if pandoc "$f" -t plain 2>/dev/null | grep -qi "viasat"; then
    echo "$f"
  fi
done

# Result: R1-2509178.docx
```

**Advanced Search**:
```bash
# Find all documents discussing "positioning"
Grep({
  pattern: "positioning|DL-TDOA|RSTD",
  glob: "*.docx",  // Won't work directly on .docx
  output_mode: "files_with_matches"
})

# Must convert first:
for f in R1-*.docx; do
  pandoc "$f" -t plain | grep -qi "DL-TDOA" && echo "$f"
done
```

**Output**: List of relevant documents

---

### **Phase 4: Deep Reading - "Extract specific content"**

**Objective**: Read full document or specific sections

**Tools Used**:
- `Bash(pandoc)` - Full document conversion
- `Bash(pandoc + grep -A/-B)` - Extract sections with context

**Reading Strategies**:

#### **A. Full Document Read**
```bash
pandoc "R1-2509178.docx" -t plain
```
- Converts entire document
- Returns full text to Claude
- Used when I need comprehensive understanding

#### **B. Section Extraction**
```bash
# Extract all "Proposal" sections
pandoc "document.docx" -t plain | grep -A 10 "^Proposal"

# Extract all "Observation" sections
pandoc "document.docx" -t plain | grep -A 5 "^Observation"
```

**Example - Viasat Document Analysis**:
```bash
pandoc "./R1-2509178.docx" -t plain
```

**Returned Content**:
```
3GPP TSG RAN WG1 #123 R1-2509178
Source: Viasat Satellite Holdings Ltd, Inmarsat

Observation 1: Always-ON synchronization signals can be used...
Proposal 1: Study UE coarse location estimation based on PSS/SSS...
```

**Output**: Structured information (proposals, observations, technical details)

---

### **Phase 5: Comparative Analysis - "How do positions differ?"**

**Objective**: Compare positions across multiple documents

**Research Pattern**:
```
1. Identify topic (e.g., "positioning approaches")
2. Find all relevant documents
3. Read each document
4. Extract position statements
5. Create comparison matrix
```

**Example from Your Folder**:

**Topic**: Positioning-based solutions

**Process**:
1. Search for positioning keywords
2. Identify 4 supporting companies: Ofinno, Airbus, IMU, Viasat
3. Read each company's document
4. Extract their proposals
5. Compare approaches:

| Company | Approach | Signals | Latency |
|---------|----------|---------|---------|
| Viasat | Single-sat PSS/SSS | Sync signals | ~40 sec |
| Airbus | Multi-sat DL-TDOA | PRS | Lower |
| Ofinno | Multi-sat DL-TDOA | PRS | Variable |
| IMU | Single-sat time-separated | PRS | Variable |

**Output**: Synthesis across documents

---

### **Phase 6: Aggregation - "What's the overall picture?"**

**Objective**: Create summaries and statistics

**Techniques**:

#### **A. Statistical Counting**
```bash
# Count documents per company
for f in R1-*.docx; do
  pandoc "$f" -t plain | head -25 | grep -i "^Source:"
done | sed 's/Source: //' | sort | uniq -c | sort -rn
```

**Output**:
```
3 Moderator (Thales)
2 Qualcomm Incorporated
2 CATT
1 Viasat Satellite Holdings Ltd, Inmarsat
...
```

#### **B. Position Categorization**
- Read all documents
- Categorize into: Pro-positioning, Anti-positioning, Neutral
- Count: 4 pro, 1 anti, 41 neutral/other

#### **C. Parameter Extraction**
From evaluation spreadsheets:
```bash
ls "Evaluation of differential delay"*.xlsx
```
Companies providing data: Samsung, Thales, Tejas, China Telecom, Fraunhofer, Eutelsat, Toyota

**Output**: Statistical summaries, aggregate views

---

## Tool Arsenal: Detailed Capabilities

### **1. Bash Tool**

**Purpose**: Execute any shell command

**Common Research Commands**:
```bash
# File listing with filters
ls -1 *.docx | grep "Qualcomm"

# Text conversion
pandoc "file.docx" -t plain

# Pattern searching
grep -i "pattern" file.txt

# Piping for complex operations
pandoc file.docx -t plain | grep -A 5 "Proposal" | head -20

# Counting
ls *.docx | wc -l

# Sorting and unique
... | sort | uniq -c | sort -rn
```

**Advantages**:
- Full shell power (pipes, redirects, loops)
- Can combine tools (pandoc + grep + head)
- Iterative processing (for loops)

**Limitations**:
- Syntax must be correct
- Security checks may require approval
- Timeouts on long operations

---

### **2. Read Tool**

**Purpose**: Direct file reading

**Usage**:
```javascript
Read({
  file_path: "/path/to/file.md",
  offset: 0,      // Optional: start line
  limit: 2000     // Optional: number of lines
})
```

**Best For**:
- Text files (.md, .txt, .json)
- Reading analysis files (companiespositioning.md)
- PDFs (auto-converts)
- Images (displays visually)

**Example from Session**:
```javascript
Read({ file_path: "/Users/sandeep/Dropbox/dev/POC/vsatRag/data/files/companiespositioning.md" })
```
Returns full markdown with line numbers.

**Limitations**:
- Cannot read binary files directly (.docx, .xlsx)
- Must use pandoc/conversion for Office files

---

### **3. Grep Tool**

**Purpose**: Fast content search using ripgrep

**Usage**:
```javascript
Grep({
  pattern: "Viasat|positioning",  // Regex pattern
  path: "/path/to/search",        // Directory or file
  glob: "*.md",                   // File filter
  output_mode: "files_with_matches", // or "content" or "count"
  "-i": true,                     // Case insensitive
  "-A": 5,                        // Lines after match
  "-B": 2                         // Lines before match
})
```

**Best For**:
- Searching text files
- Finding all occurrences
- Getting context around matches

**Example**:
```javascript
Grep({
  pattern: "Proposal.*positioning",
  glob: "*.md",
  output_mode: "content",
  "-A": 3
})
```

**Limitations**:
- Doesn't work on binary files (.docx)
- Must convert first with pandoc

---

### **4. Glob Tool**

**Purpose**: Find files by pattern

**Usage**:
```javascript
Glob({
  pattern: "**/*.docx"  // Recursive search
})

Glob({
  pattern: "R1-2509*.docx"  // Specific range
})

Glob({
  pattern: "Evaluation*.xlsx"  // All evaluation files
})
```

**Best For**:
- File discovery
- Pattern-based searches
- Building file lists

**Example Output**:
```
/path/R1-2509131.docx
/path/R1-2509178.docx
...
```

---

### **5. Task Tool (Subagents)**

**Purpose**: Launch specialized research agents

**Agent Types Available**:

#### **Explore Agent**
```javascript
Task({
  subagent_type: "Explore",
  prompt: "Find all documents from Qualcomm and summarize their position on positioning-based solutions",
  description: "Explore Qualcomm position"
})
```

**What it does**:
- Autonomous multi-step search
- Reads multiple files
- Synthesizes findings
- Returns comprehensive report

**When to use**:
- Complex, multi-document research
- "Find everything about X"
- Uncertain what files contain info

#### **General-Purpose Agent**
```javascript
Task({
  subagent_type: "general-purpose",
  prompt: "Create a comparison table of all companies' positions on frequency control",
  description: "Compare frequency positions"
})
```

**What it does**:
- Multi-step autonomous research
- Can read, search, analyze
- Creates structured outputs

---

## Research Patterns & Strategies

### **Pattern 1: Company Position Analysis**

**Goal**: Understand a specific company's stance

**Steps**:
1. Find company's documents (metadata extraction)
2. Read each document fully
3. Extract proposals and observations
4. Categorize technical approach
5. Note supporting evidence

**Tools Sequence**:
```
Bash(for loop + grep "Source:") → Identify docs
   ↓
Bash(pandoc full document) → Read content
   ↓
Extract proposals/observations → Parse structure
   ↓
Categorize approach → Analyze
```

---

### **Pattern 2: Cross-Document Topic Search**

**Goal**: Find all mentions of a topic across documents

**Steps**:
1. Define search keywords
2. Search all documents
3. Extract relevant sections
4. Aggregate findings

**Tools Sequence**:
```
Glob("*.docx") → Get file list
   ↓
Bash(for each: pandoc + grep) → Search each
   ↓
Bash(pandoc + grep -A 10) → Extract sections
   ↓
Aggregate results → Synthesize
```

---

### **Pattern 3: Statistical Analysis**

**Goal**: Generate statistics about the document set

**Steps**:
1. Extract metadata from all docs
2. Count/categorize
3. Calculate percentages
4. Create summary tables

**Tools Sequence**:
```
Bash(for loop + head + grep) → Extract metadata
   ↓
Bash(sort | uniq -c) → Count occurrences
   ↓
Calculate percentages → Analyze
   ↓
Create tables → Present
```

---

### **Pattern 4: Parameter Extraction**

**Goal**: Extract specific numerical/technical parameters

**Steps**:
1. Identify documents with parameters
2. Read relevant sections
3. Extract values
4. Tabulate

**Example - Evaluation Parameters**:
```bash
# Find evaluation spreadsheets
ls "Evaluation*.xlsx"

# Extract company names from filenames
ls "Evaluation*.xlsx" | sed 's/.*_//' | sed 's/.xlsx//'
```

**From document text**:
```bash
pandoc doc.docx -t plain | grep -E "LEO-[0-9]+" -A 5
```

---

## Real-World Research Example: Viasat Analysis

### **User Question**: "Explain role of Viasat"

### **My Research Process**:

**Step 1**: Initial search (failed)
```
Grep("Viasat") → No results (doesn't work on .docx)
```

**Step 2**: Convert and search
```
Bash(for f in R1-*.docx; do pandoc "$f" -t plain | grep -qi "viasat" && echo "$f"; done)
→ Found: R1-2509178.docx
```

**Step 3**: Read full document
```
Bash(pandoc "./R1-2509178.docx" -t plain)
→ Returns full text
```

**Step 4**: Extract key information
- Source: Viasat + Inmarsat
- Approach: PSS/SSS-based positioning
- Observations: 2
- Proposals: 3
- Latency: ~40 seconds

**Step 5**: Analyze and synthesize
- Compare to other positioning advocates
- Identify unique approach (PSS/SSS vs PRS)
- Note trade-offs (latency vs overhead)

**Step 6**: Answer user
- Viasat's position explained
- Comparison to peers
- Technical details

---

## Advanced Capabilities

### **1. Parallel Research**

I can research multiple topics simultaneously:

```javascript
// Launch 3 searches in parallel
await Promise.all([
  Bash("pandoc doc1.docx -t plain"),
  Bash("pandoc doc2.docx -t plain"),
  Bash("pandoc doc3.docx -t plain")
])
```

**Benefit**: Faster research

---

### **2. Iterative Refinement**

**Pattern**:
```
Initial broad search → Too many results
   ↓
Refine search terms → Better targeting
   ↓
Extract specific sections → Precise data
```

**Example**:
```
Search "positioning" → 20 documents
   ↓
Search "DL-TDOA positioning" → 4 documents
   ↓
Extract "Proposal.*DL-TDOA" → Exact proposals
```

---

### **3. Multi-Format Handling**

**Document Types I Can Process**:

| Format | Tool | Method |
|--------|------|--------|
| .md | Read | Direct |
| .txt | Read | Direct |
| .docx | Bash(pandoc) | Convert to text |
| .pdf | Read | Auto-convert |
| .xlsx | Limited | File list only (not content) |
| .json | Read | Direct |
| Images | Read | Visual display |

---

### **4. Context-Aware Analysis**

**I use CLAUDE.md** for domain knowledge:

From `/Users/sandeep/Dropbox/dev/POC/vsatRag/CLAUDE.md`:
- Document naming: R1-XXXXXXX format
- Source field location: first 20 lines
- Key terms: DL-TDOA, PRACH, TA, Doppler
- Meeting context: RAN WG1 #123

This guides my research strategy.

---

## Limitations & Workarounds

### **Limitation 1**: Binary File Reading

**Problem**: Cannot read .docx/.xlsx directly

**Workaround**: Use pandoc conversion
```bash
pandoc "file.docx" -t plain
```

---

### **Limitation 2**: Large File Handling

**Problem**: Token limits on huge files

**Workaround**:
- Read in chunks (offset/limit)
- Extract specific sections only
- Use grep to filter before reading

---

### **Limitation 3**: Excel Data Extraction

**Problem**: Cannot parse .xlsx content

**Workaround**:
- Metadata from filename
- Convert to CSV if needed
- Focus on .docx for technical content

---

### **Limitation 4**: Performance on Many Files

**Problem**: 45+ files = many operations

**Workaround**:
- Use for loops in Bash (efficient)
- Parallel operations when possible
- Cache results mentally in conversation

---

## Summary: Research Workflow

```
┌─────────────────────────────────────────┐
│          User Question                  │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  1. Discovery (ls, glob)                │
│     → What files exist?                 │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  2. Metadata (pandoc + head + grep)     │
│     → Who created what?                 │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  3. Search (grep, pandoc + grep)        │
│     → Where is relevant info?           │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  4. Deep Read (pandoc, Read)            │
│     → Extract specific content          │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  5. Analysis (LLM reasoning)            │
│     → Synthesize findings               │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  6. Answer (natural language)           │
│     → Respond to user                   │
└─────────────────────────────────────────┘
```

---

## Key Insight

**Claude Code doesn't "know" what's in documents** - it actively researches them in real-time using:
- File system tools (ls, find, grep)
- Conversion tools (pandoc)
- Reading tools (Read, Bash)
- Analysis tools (LLM reasoning)

The research is **dynamic, adaptive, and tool-driven** - just like a human researcher would use command-line tools to explore a document repository.
