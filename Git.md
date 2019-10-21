# Popular science: Git

## Concept

### Distributed VCS

|       | Git             | SVN                    |
|:------|:----------------|:-----------------------|
| Model | **Distributed** | **Centralized**        |
| Repo  | Remote & Local  | Central & Working copy |

Git advantage:

- No discrimination between remote & local
- Do everything offline
- No single point of failure

### Architecture

1. Repository
    - Contain the project
    - 2 components:
        - Working tree: Current state
        - `.git` folder: History in blobs
    - Remote & local:
        - Remote: hosted elsewhere
            - Enable collaboration
        - Local: right here
        - `origin`? Just default remote name
    - `git remote rm/add/...`

2. Branch
    - Development path
    - `master`? Just default branch name
    - `git branch name`: create new branch
        - Initially a clone of the current branch
    - `git checkout name`: move to new branch



3. Staging area
    - Separate working tree from repo
        - Allow selective commit
    - Example: Machine-specific config
        - If `commit` saves everything: duplicated config
        - Need selective commit => only commit **staged** changes
    - Common flow:
        1. `git add`: stage *needed* files
        2. `git commit`
        3. `git push`

4. Commit
    - Save the current modification
    - Internal: save modification in blob, return SHA-1
        - Another aspect of `git`: **key-value DB**!

5. `HEAD` & `head` pointer
    - `HEAD` points branch
    - `head` points commit
    - Pointers are powerful tool to navigate in git
    - More on pointer later
