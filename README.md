# MiniGit

![Python Version](https://img.shields.io/badge/python-3.12+-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![Code Size](https://img.shields.io/github/languages/code-size/mann-127/MiniGit)
[![GitHub Stars](https://img.shields.io/github/stars/mann-127/MiniGit?style=social)](https://github.com/mann-127/MiniGit)

**A minimal Git implementation in pure Python that speaks Git's native object format.**

MiniGit is an educational project that reimplements Git's core plumbing commands from scratch. It demonstrates how Git's content-addressable storage works under the hood—turning files into blobs, directories into trees, and snapshots into commits. Every object MiniGit creates is 100% compatible with Git's storage format, which you can verify by cloning a real repository and inspecting its objects.

---

## Quick Start

```bash
# Clone and try it in 3 commands
git clone https://github.com/mann-127/MiniGit.git && cd MiniGit
python -m minigit.cli init
echo "Hello Git!" > test.txt && python -m minigit.cli hash-object -w test.txt
```

**Output**: `557db03de997c86a4a028e1ebd3a1ceb225be238` (SHA-1 hash of your file)

---

## Features

| Command                                                 | Description                                                                              |
| ------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `init`                                                  | Bootstrap a `.git/` directory with default references.                                   |
| `hash-object -w <file>`                                 | Store a file as a **blob** object and print its SHA‑1.                                   |
| `cat-file -p <sha1>`                                    | Pretty‑print the contents of a blob.                                                     |
| `write-tree`                                            | Create a **tree** object that captures the current working directory (excluding `.git`). |
| `ls-tree [--name-only] <sha1>`                          | List the entries inside a tree object.                                                   |
| `commit-tree <tree_sha> [-p <parent_sha>] -m "message"` | Create a **commit** object that points to a tree and an optional parent.                 |
| `clone <url> <dir>`                                     | Clone a public repository into `<dir>` (internally calls `git clone`).                   |

---

## Why MiniGit?

- **Learn by doing**: See exactly how Git stores objects, computes SHA-1 hashes, and builds the commit graph
- **Zero dependencies**: Built entirely with Python's standard library (`hashlib`, `zlib`, `pathlib`, `time`)
- **Git-compatible**: Objects created by MiniGit can be read by Git, and vice versa
- **Clean codebase**: **~280 lines** of readable Python vs. Git's **~300,000 lines of C**
- **Educational focus**: Perfect for understanding Git internals without the complexity

---

## Architecture Overview

```
Working Directory
    ├─ file.txt ──────────────► Blob (content-addressed by SHA-1)
    ├─ src/
    │   └─ main.py ───────────► Blob
    └─ docs/
        └─ README.md ─────────► Blob
                ↓
            Tree Object (snapshot of directory structure)
                ↓
            Commit Object (points to tree + metadata)
                ↓
            Parent Commit(s) (history chain)
```

**Git's Object Model:**
- **Blobs**: Store file contents
- **Trees**: Store directory listings (like a filesystem snapshot)
- **Commits**: Store trees + metadata (author, message, timestamp, parents)

---

## What It Does

MiniGit implements seven fundamental Git operations:

### Repository Initialization
```bash
python -m minigit.cli init
```
Creates a `.git` directory with the standard object storage structure and references.

### Blob Operations
```bash
# Store a file as a blob object
python -m minigit.cli hash-object -w myfile.txt

# Read a blob's contents
python -m minigit.cli cat-file -p <sha1>
```

### Tree Operations
```bash
# Snapshot the entire working directory as a tree
python -m minigit.cli write-tree

# Inspect a tree object's entries
python -m minigit.cli ls-tree <tree_sha>
python -m minigit.cli ls-tree --name-only <tree_sha>
```

### Commit Operations
```bash
# Create a commit pointing to a tree
python -m minigit.cli commit-tree <tree_sha> -m "Initial commit"

# Create a commit with a parent (history chaining)
python -m minigit.cli commit-tree <tree_sha> -p <parent_sha> -m "Second commit"
```

### Clone (Git Delegation)
```bash
# Clone a repository using system Git
python -m minigit.cli clone https://github.com/user/repo target-dir
```
*Note: This command delegates to your system's Git binary—MiniGit doesn't yet implement the Smart HTTP protocol.*

---

## How It Works

### Content-Addressable Storage
Every object (blob, tree, or commit) is identified by the SHA-1 hash of its contents. Files are stored in `.git/objects/` using a fan-out directory structure: the first two characters of the hash become a subdirectory, and the remaining 38 characters are the filename.

**Example**: The SHA-1 `a1b2c3d4...` is stored at `.git/objects/a1/b2c3d4...`

### Object Format
All objects follow Git's native format:
```
<type> <size>\0<content>
```
This is then compressed with zlib before being written to disk.

### Tree Building
The `write-tree` command recursively walks your working directory:
1. Files become **blob objects** (with SHA-1 hashes of their content)
2. Directories become **tree objects** (lists of mode, name, and SHA-1 for each entry)
3. Trees reference other trees (subdirectories) or blobs (files)
4. The root tree represents your entire project snapshot

### Commit Structure
Commits are text objects that contain:
- A pointer to a tree (the snapshot)
- Zero or more parent commits (for history)
- Author and committer metadata (name, email, timestamp, timezone)
- A commit message

---

## Requirements

- **Python 3.12+** (uses modern type hints like `Path | None`)
- **Git** (only required for the `clone` command)

---

## Installation

### From Source
```bash
git clone https://github.com/mann-127/MiniGit.git
cd MiniGit
python -m minigit.cli init
```

### Optional: Install in Editable Mode
```bash
pip install -e .
```

---

## Project Structure

```
MiniGit/
├── minigit/
│   ├── __init__.py
│   └── cli.py           # All commands and core logic (~280 lines)
├── pyproject.toml       # Python project metadata
├── README.md
├── LICENSE
└── .gitignore
```

---

## Example Workflow

```bash
# Initialize a new repository
python -m minigit.cli init

# Create some files
echo "Hello, world!" > hello.txt
mkdir src
echo "print('test')" > src/main.py

# Store files as blobs
python -m minigit.cli hash-object -w hello.txt
python -m minigit.cli hash-object -w src/main.py

# Snapshot the working directory
tree_sha=$(python -m minigit.cli write-tree)

# Create a commit
commit_sha=$(python -m minigit.cli commit-tree $tree_sha -m "Initial commit")

# Verify with real Git
git cat-file -p $commit_sha
git cat-file -p $tree_sha
```

---

## Common Pitfalls

⚠️ **Command not found?**  
Run commands as `python -m minigit.cli <command>`, not `minigit <command>`

⚠️ **Python version error?**  
MiniGit requires Python 3.12+ for modern type hints

⚠️ **Clone doesn't work?**  
The `clone` command delegates to system Git—ensure Git is installed

⚠️ **Where's `git add`?**  
MiniGit has no staging area—it works directly with the working directory

---

## What's Missing (By Design)

MiniGit focuses on the **object model** and intentionally omits:
- The staging area (index)
- Branch operations and HEAD manipulation
- Merge logic
- Packfiles and deltification
- Smart HTTP protocol for network operations
- Ref management beyond initialization

These omissions keep the codebase small and focused on Git's core storage concepts.

---

## Future Possibilities

- [ ] Implement a pure-Python packfile parser to eliminate the Git dependency for `clone`
- [ ] Add staging area support (`add`, `reset`)
- [ ] Handle merge commits (multiple parents)
- [ ] Implement `checkout` to restore working directory from commits
- [ ] Add branch creation and switching
- [ ] Support for tags and refs
- [ ] Basic diff implementation

---

## Verification

You can prove MiniGit's compatibility by:

1. Using `git cat-file` to read MiniGit's objects
2. Using MiniGit to read objects created by Git
3. Cloning a real repository and inspecting it with MiniGit's commands

```bash
python -m minigit.cli clone https://github.com/git/git test-repo
cd test-repo
python -m minigit.cli ls-tree HEAD^{tree}  # Read Git's native objects
```

---

## Learning Resources

To deepen your understanding of what MiniGit implements:

- 📖 [Git Internals - Git Objects (Pro Git Book)](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)
- 🛠️ [Write Yourself a Git](https://wyag.thb.lt/)
- 📚 [Building Git (book by James Coglan)](https://shop.jcoglan.com/building-git/)
- 🧠 [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/)

---

## Comparison with Other Git Implementations

| Feature | MiniGit | ugit | pygit2 | Git (C) |
|---------|---------|------|--------|---------|
| **Lines of Code** | ~280 | ~500 | Bindings | ~300K |
| **Dependencies** | None | None | libgit2 | Many |
| **Object Format** | Native Git ✅ | Native Git ✅ | Native Git ✅ | Native Git ✅ |
| **Purpose** | Education | Education | Production | Production |
| **Index/Staging** | ❌ | ✅ | ✅ | ✅ |
| **Network Clone** | Delegates to Git | ❌ | ✅ | ✅ |

---

## Contributing

This is an educational project, but contributions are welcome! If you'd like to:
- Add new commands
- Improve documentation
- Fix bugs
- Add tests
- Enhance error handling

Feel free to open an issue or pull request.

---

## License

MIT License. See `LICENSE` for details.

---

## Author

**mann-127**
- GitHub: [@mann-127](https://github.com/mann-127)
- Repository: [MiniGit](https://github.com/mann-127/MiniGit)

---

## Acknowledgments

- Inspired by the Git community's commitment to open-source education
- Built upon the excellent documentation in the [Pro Git Book](https://git-scm.com/book)
- Thanks to James Coglan's ["Building Git"](https://shop.jcoglan.com/building-git/) for implementation insights

---

**Built to understand Git, one object at a time.** ⚡
