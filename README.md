# MiniGit

**A minimal Git implementation in pure Python that speaks Git's native object format.**

MiniGit is an educational project that reimplements Git's core plumbing commands from scratch. It demonstrates how Git's content-addressable storage works under the hood—turning files into blobs, directories into trees, and snapshots into commits. Every object MiniGit creates is 100% compatible with Git's storage format, which you can verify by cloning a real repository and inspecting its objects.

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

## Why MiniGit?

- **Learn by doing**: See exactly how Git stores objects, computes SHA-1 hashes, and builds the commit graph
- **Zero dependencies**: Built entirely with Python's standard library (`hashlib`, `zlib`, `pathlib`, `time`)
- **Git-compatible**: Objects created by MiniGit can be read by Git, and vice versa
- **Clean codebase**: ~280 lines of readable Python that implement the core Git data model

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

## Requirements

- **Python 3.12+** (uses modern type hints like `Path | None`)
- **Git** (only required for the `clone` command)

## Installation

Clone this repository and run commands directly:

```bash
git clone https://github.com/mann-127/MiniGit.git
cd MiniGit
python -m minigit.cli init
```

## Project Structure

```
MiniGit/
├── minigit/
│   ├── __init__.py
│   └── cli.py           # All commands and core logic
├── pyproject.toml       # Python project metadata
├── README.md
├── LICENSE
└── .gitignore
```

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

## What's Missing (By Design)

MiniGit focuses on the **object model** and intentionally omits:
- The staging area (index)
- Branch operations and HEAD manipulation
- Merge logic
- Packfiles and deltification
- Smart HTTP protocol for network operations
- Ref management beyond initialization

These omissions keep the codebase small and focused on Git's core storage concepts.

## Future Possibilities

- Implement a pure-Python packfile parser to eliminate the Git dependency for `clone`
- Add staging area support (`add`, `reset`)
- Handle merge commits (multiple parents)
- Implement `checkout` to restore working directory from commits
- Add branch creation and switching

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

## License

MIT License. See `LICENSE` for details.

## Contributing

This is an educational project, but contributions are welcome! If you'd like to:
- Add new commands
- Improve documentation
- Fix bugs
- Add tests

Feel free to open an issue or pull request.

---

**Built to understand Git, one object at a time.**
