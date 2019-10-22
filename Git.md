# Popular science: Git

[https://imgs.xkcd.com/comics/git.png]

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
    - = project
    - 2 components:
        - Working tree: Current state
        - `.git` folder: History in blobs
    - Remote & local:
        - Remote: hosted elsewhere
            - Enable collaboration
        - Local: right here
        - `origin`? Just default remote name
    - `git remote rm/add/...`

2. Staging area
    - Separate working tree from repo
        - Allow selective commit
    - Example: Machine-specific config
        - If `commit` saves everything: duplicated config
        - Need selective commit => only commit **staged** changes
    - Common flow:
        1. `git add`: stage *needed* files
        2. `git commit`
        3. `git push`

3. Commit
    - Save the current modification
        - Analogy: game checkpoint
        - ID: SHA-1
    - Internal:
        - Save in blobs
        - Another aspect of git: **key-value DB**!

4. Branch
    - Development path
    - `master`? Just default branch name
        - Also called *trunk*
    - `git branch name`: create new branch
        - Initially a clone of the current branch
    - `git checkout name`: move to branch

5. Fork
    - Also development path lmao
    - Compare to branch

    |        | Fork        | Branch      |
    |:-------|:------------|:------------|
    | Intent | New product | New feature |
    | Output | Repo        | Branch      |
    | Merge  | Won't       | Will        |
    | Tool   | git server  | git         |

6. Pointers
    - **Branch points commit!**
    - `HEAD` points branch
    - `head` points commit
    - Pointers are powerful tool to navigate in git
    - More on pointer later

## Git merge

- Q: Branched. What do?
- A: Merge!

[https://imgur.com/gallery/gsDdNVy]

1. Basic merging: 2 methods

    1. Merge **B into A**
        Incoporate changes directly

        ```bash
        git checkout master
        git merge fix
        # look ma no conflict
        ```

        ```mermaid
        graph BT
            subgraph after
                C1a((C1)) --> C0a((C0));
                C2a((C2)) --> C1a;
                C3a((C3)) --> C1a;

                C4a((C4)) --> C3a;
                C4a --> C2a;

                fa(fix) --> C3a;
                ma(master) --> C4a;
            end

            subgraph before
                C1b((C1)) --> C0b((C0));
                C2b((C2)) --> C1b;
                C3b((C3)) --> C1b;

                mb(master) --> C2b;
                fb(fix) --> C3b;
            end
        ```

    2. Rebase: 2nd type of merging
        Move the branch's starting point to another commit

        ```bash
        git checkout fix
        git rebase master
        ```

        ```mermaid
        graph BT
            classDef dashed stroke-dasharray: 5, 5;
            classDef master fill: #176655;
            classDef fix fill: #484A4B;

            subgraph after
                C1a((C1)) --> C0a((C0));
                C2a((C2)) --> C1a;
                C3a((C3)) -.-> C1a;
                C4a((C4)) -.-> C3a;

                C3a_((C3')) --> C2a;
                C4a_((C4')) --> C3a_;

                ma(master) --> C2a;
                fa(fix) --> C4a_;

                class C3a dashed;
                class C4a dashed;
                class ma master;
                class fa fix;
            end

            subgraph before
                C1b((C1)) --> C0b((C0))
                C2b((C2)) --> C1b
                C3b((C3)) --> C1b
                C4b((C4)) --> C3b

                mb(master) --> C2b
                fb(fix) --> C4b

                class mb master;
                class fb fix;
            end
        ```
