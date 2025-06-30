# MiniGit
A minimal Git engine: serializes files into blobs, directories into trees, snapshots those into commits, and by delegating cloning to real Git, proves its objects are 100 % compatible with Git’s storage model.

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

All objects are stored in Git’s native format: `zlib`‑compressed, SHA‑1 addressed, and laid out in the fan‑out structure (`.git/objects/aa/bb…`).

---

## Installation

MiniGit requires **Python ≥ 3.12** and a working **Git** executable in your `PATH` (only the `clone` command relies on Git).

```
MiniGit/
├── minigit/
│   ├── __init__.py
│   └── cli.py        
├── pyproject.toml
├── README.md
├── LICENSE           
└── .gitignore
```

---

## Design Highlights

- **Content‑addressable storage** – every object’s filename is its SHA‑1 hash of header + content.
- **Fan‑out layout** – objects live at `.git/objects/<aa>/<rest>` to avoid large directory fan‑ins.
- **Pure stdlib** – hashing (`hashlib`), compression (`zlib`), paths (`pathlib`), and timestamps (`time`) are the only requirements.
- **Recursive tree builder** – traverses the working directory, converts files to blobs, folders to trees, and returns the root tree SHA.
- **Commit serializer** – produces valid commit objects with author/committer metadata, timestamps, and parent linkage.
- **System‑Git delegation** – the `clone` command shells out to `git clone` so MiniGit can fetch real repositories without re‑implementing Smart HTTP and packfile parsing.

---

## Roadmap / Ideas

- Replace the cloned delegation with a pure‑Python Smart HTTP + packfile parser.
- Add support for the Git index (staging area) and implement `add`.
- Multiple parents in `commit-tree` to support merge commits.
- `checkout` and branch manipulation.

Contributions are welcome – open an issue or pull request!

---

## License

MiniGit is licensed under the **MIT License**.  See the `LICENSE` file for details.
