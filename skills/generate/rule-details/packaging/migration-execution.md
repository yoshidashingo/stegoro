# Migration Execution — PACKAGING Phase

## Purpose
Migrate improved policy files from legacy directory structure to the new plugin structure. This stage handles the transition from older formats (e.g., `.steering/`) to the standardized plugin layout.

## Prerequisites
- Plugin Structure Generation (P1) completed and approved
- Automated Validation (P2) PASS
- Legacy policy structure exists and is identified

## Execution Classification
**Type**: CONDITIONAL

**Execute IF**:
- Existing policy brush-up (legacy structure exists, e.g., `.steering/` directory)
- User confirmed migration is desired during Scope Definition

**Skip IF**:
- New policy generation (no migration source exists)
- User explicitly opted out of migration

---

## Execution Steps

### Step 1: Identify Migration Source
**Action**: Locate and catalog legacy policy structure
**Input**: Repository file system
**Output**: Legacy file inventory

1. Scan for legacy directories (`.steering/`, `.<old-agent-name>/`, etc.)
2. List all files with paths and line counts
3. Identify which files have been improved during GENERATION/REFINEMENT
4. Flag files that exist in legacy but NOT in the new plugin structure

### Step 2: Create File Path Mapping
**Action**: Map old paths to new paths
**Input**: Legacy inventory + plugin structure
**Output**: Migration mapping table

```markdown
## Migration Mapping

| # | Legacy Path | New Path | Action |
|---|------------|----------|--------|
| 1 | .steering/aws-aidlc-rules/core-workflow.md | <agent>/rules/core-workflow.md | Copy (improved) |
| 2 | .steering/aws-aidlc-rule-details/common/*.md | <agent>/<agent>-rule-details/common/*.md | Copy (improved) |
| 3 | .steering/aws-aidlc-rule-details/<phase>/*.md | <agent>/<agent>-rule-details/<phase>/*.md | Copy (improved) |
| ... | ... | ... | ... |
```

**Present mapping to user for approval before executing.**

### Step 3: Check for Source Changes
**Action**: Detect if legacy files have been modified since GENERATION started
**Input**: Legacy files, git status
**Output**: Change detection report

1. Compare legacy file timestamps or content hashes with GENERATION start time
2. If changes detected:
   - **Severity**: Medium
   - Present diff to user
   - Ask: "Manual merge or overwrite with improved version?"
   - **Do Not Proceed** until user confirms strategy

### Step 4: Execute Copy
**Action**: Copy improved files to plugin structure
**Input**: Approved mapping + improved files
**Output**: Populated plugin directory

For each file in the mapping:
1. Copy improved version to new path
2. Verify copy integrity (file exists, content matches)
3. Log each copy operation

### Step 5: Update References
**Action**: Update all external references to use new paths
**Input**: Repository files that reference the legacy structure
**Output**: Updated references

Files to check and update:
- `CLAUDE.md` — routing rules, directory structure documentation
- `.claude/rules/routing-*.md` — Glob-based routing rules
- `README.md` / `README-ja.md` — documentation references
- Any other files referencing legacy paths

**Method**:
1. Grep for all legacy path patterns across the repository
2. Update each reference to the new plugin path
3. Verify no legacy path references remain

### Step 6: Mark Legacy as Deprecated
**Action**: Add deprecation notice to legacy directory
**Input**: Legacy directory
**Output**: Deprecated legacy with notice

**CRITICAL**: Do NOT delete legacy files. Mark as deprecated only.

Add `DEPRECATED.md` to the legacy directory root:

```markdown
# DEPRECATED

This directory has been superseded by the `<agent-name>/` plugin structure.

- **Migration date**: [ISO timestamp]
- **New location**: `<agent-name>/`
- **Action**: Use the new plugin structure. This directory is preserved for reference only.
- **Safe to delete**: After confirming the new structure works correctly.
```

### Step 7: Present Migration Report
**Action**: Summarize migration results for user approval
**Input**: All migration operations
**Output**: Migration report

```markdown
## Migration Report

### Files Migrated
- Total files: [N]
- Successfully copied: [N]
- Skipped (no change): [N]

### References Updated
- Files checked: [N]
- References updated: [N]
- Legacy path references remaining: [N] (should be 0)

### Legacy Status
- Directory: [legacy path]
- Status: Deprecated (DEPRECATED.md added)
- Action required: None (safe to delete after verification)
```

---

## Output Artifacts

### Migration Report
- **File**: `steering-docs/packaging/migration-report.md`
- **Content**: File mapping, copy results, reference updates

### DEPRECATED.md
- **File**: `<legacy-directory>/DEPRECATED.md`
- **Content**: Deprecation notice with migration details

---

## Completion Message

### REVIEW REQUIRED

**Migration Execution is complete.**

- **Files Migrated**: [N] files to `<agent-name>/`
- **References Updated**: [N] references across [N] files
- **Legacy Status**: Deprecated (not deleted)

### WHAT'S NEXT

The steering policy workflow is complete. Your plugin is ready for use.

1. Review the generated plugin in `<agent-name>/`
2. Install and test with a sample task
3. Provide feedback for iteration if needed

---

## Error Handling

### Source Directory Changes Detected
- **Severity**: Medium
- **Solution**: Present diff to user, ask for merge strategy (manual merge / overwrite)
- **Do Not Proceed**: Until user confirms

### Reference Path Update Missed
- **Severity**: High
- **Solution**: Re-grep entire repository for legacy path patterns, fix all occurrences
- **Recovery**: Run comprehensive path search after initial update

### Legacy Deletion Request
- **Severity**: Low
- **Solution**: Advise "deprecation only, deletion is user's decision after verification"
- **Recovery**: If user insists, confirm they have verified the new structure works

## References
- `common/implementation-knowhow.md` — Migration guide, path mapping patterns
- `discovery/scope-definition.md` — Plugin structure design
