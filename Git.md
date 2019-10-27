# Popular science: Git

![Git intro](https://imgs.xkcd.com/comics/git.png)

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
        - Related: `.gitignore`
    - **Common flow**:
        1. `git add`: stage *needed* files
        2. `git commit`
        3. `git push`

3. Commit
    - Current snapshot
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

5. Pointers
    - **Branch points commit!**
    - `HEAD` points branch/commit
    - Pointers are powerful tool to navigate in git
    - More on pointer later

6. Commit tree
    - Git is all about **commit**
    - Commit = tree node
    - *tree* fits the analogy
        - DAG in fact

    ```mermaid
    graph BT
        classDef master fill:#418a44;
        classDef fix fill:#484A4B;

        C1((C1)) --> C0((C0));
        C2((C2)) --> C1;
        C3((C3)) --> C1;

        mb("->master<-") --> C2;
        fb(fix) --> C3;

        class mb master
        class fb fix
    ```

7. Fork
    - Also development path lmao
    - Compare to branch

    |        | Fork        | Branch      |
    |:-------|:------------|:------------|
    | Intent | New product | New feature |
    | Output | Repo        | Branch      |
    | Merge  | Won't       | Will        |
    | Tool   | git server  | git         |

## Git navigation

1. Refer to a commit
    5 interchangeable methods:

    1. Raw ID
        - Remind: commit has unique ID

        ```bash
        # Get info, content, diff,...
        git show commit_pointer
        # Get ID
        git rev-parse commit_pointer
        ```

    2. Branch
        - Remind: branch = commit pointer
            - Default: points at the **top**
        - `git branch`
            - `new_name`: new branch
            - `-m new_name`: rename current
            - `-l`: list
            - `-a`: list remote branch
            - `-d`: delete *merged* branch
            - `-D`: delete any branch
        - Branch can points at non-top commit
            - See `git checkout`

    3. `HEAD` & special pointers
        - `HEAD`: points *current checked-out*
            - Usually is a branch
            - Can also be a commit -> detached `HEAD`
        - More on `checkout` later
        - Etym: top of the branch
        - One of special pointers
            - git handles what is pointed
            - E.g.: `REVERT_HEAD`, `CHERRY_PICK_HEAD`,...

        ```bash
        # Move HEAD to sth
        git checkout sth
        ```

    4. Tag
        - Still pointers!
        - Difference:
            - Branch/`HEAD`: move with commits
            - Tags: fixed point
        - Usually used to mark version

        ```bash
        # lightweight - no message
        git tag tag_name
        # annotated - have message
        git tag tag_name -m "tag message"
        ```

    5. Relative ref
        - Raw pointers are dirty -> Meet relative ref
        - `~` & `^`: back reference suffix
        - Similarity:
            - Go with number: `master~5`
            - Chainable: `HEAD^~^`
        - `~n`: Go up nth generation, at the first parent
            - For dummies: Go up **ancestor**
            - `~` == `~1`
            - `~2`: first parent's first parent
        - `^n`: Go up to the nth parent (if > 1 parents)
            - For dummies: Go up **parent**
            - `^` == `^1`
            - `^3`: third parent
            - Equal `~` if **no** merge commit along the path
            - Cause: DAG != tree

        ```mermaid
        graph BT
            classDef master fill:#418a44;

            subgraph ref from HEAD
                C1((C1));
                C2((C2)) -- "HEAD~4" --> C1;
                C3((C3)) -- "HEAD~^2~~" --> C1;
                C4((C4)) -- "HEAD~^2~" --> C3;
                C5((C5)) -- "HEAD~^" --> C2;
                C6((C6)) -- "HEAD~^2" --> C4;
                C6 -- "HEAD~~" --> C5;
                C7((C7)) -- "HEAD~1" --> C6;
                m("->master<-") --> C7;

                class m master;
            end
        ```

    6. Refspec
        - Example: `origin/master`
        - More on remote & refspec later

2. Navigation utilities
    1. `git status`
        - Show current `HEAD`, branch
        - Show staged file
        - Show file addition/deletion/modification
        - ...

    2. `git log`
        - Has many format/beautify mode
        - Famous one-liner:

            ```bash
            git log --graph --decorate --pretty=oneline --abbrev-commit

            * ddbd10c001 (HEAD, tag: 4.1.1, m, b) release: OpenCV 4.1.1
            *   7c0a43d425 Merge tag '4.1.1-openvino'
            |\  
            | * 693877212d (tag: 4.1.1-openvino) Fixed video writer filename check for plugins
            | * db211446f3 OpenCV version '-openvino'
            * |   0cf479dd5c Merge remote-tracking branch 'upstream/3.4' into merge-3.4
            |\ \  
            ```

3. Move to a commit
    1. `git checkout`
        - Checkout sth: move **working tree** to it
            - `git checkout fix`
            - `git checkout DEADB33`
        - Etym: from convention
            - `check in`: store into VCS
            - `check out`: get from VCS
        - Checkout is associated with `HEAD`
        - Checkout branch: safe
        - Checkout non-branch - detached `HEAD`!
            - NOT safe to commit, OK if not
            - Check out top commit,... also detach `HEAD`
        - Why unsafe to commit?
            - Who can see the commit then?

            ```mermaid
            graph BT
                classDef dashed stroke-dasharray: 5, 5;

                subgraph checkout branch
                    C1a((C1));
                    C2a((C2)) --> C1a;
                    C3a((C3)) --> C2a;
                    ba("->branch<-") --> C2a;

                    comment(C3 is unreachable!) -.-> C3a

                    class comment dashed
                end

                subgraph commit
                    C1c((C1));
                    C2c((C2)) --> C1c;
                    C3c(("->C3<-")) --> C2c;
                    bc(branch) --> C2c;
                end

                subgraph move up
                    C1m((C1));
                    C2m(("->C2<-")) --> C1m;
                    bm(branch) --> C2m;
                end

                subgraph initial
                    C1b((C1));
                    C2b((C2)) --> C1b;
                    bb("->branch<-") --> C2b;
                end
            ```

    2. `git revert`
        - Undo changes made by *one* commit
        - Create a new commit that undoes the commit's change

    3. `git reset`
        - Reset working tree/stage/commit tree to earlier state
            - May include delete!
        - Much more powerful & dangerous than `checkout`
        - `git reset reset_mode commit_reset_to`
        - Commit to reset to: default `HEAD`
        - 3 reset modes

            |             | `--soft` | `--mixed` (default)  | `--hard`      |
            |:------------|:---------|:---------------------|:--------------|
            | Working dir | Keep change  | Same as left     | Delete change |
            | Stage area  | Stage change | Not stage change | Same as left  |
            | Commit tree | Remove to commit | Same as left | Same as left  |

            ![Modes of git reset](https://wac-cdn.atlassian.com/dam/jcr:7fb4b5f7-a2cd-4cb7-9a32-456202499922/03%20(8).svg?cdnVersion=649)

    4. `git reset` vs `git checkout`
        Key takeaway:
        - `git checkout`: Change **working tree**
        - `git revert`: Undo one commit
        - `git reset`: Reset **stage**

## Git merge

- Q: Branched. Wat do?
- A: Merge!

![Thanh's first merge](https://i.imgur.com/gsDdNVy.jpg)

1. Basic merging: 2 methods

    1. Merge
        Incoporate changes directly

        ```bash
        # Merge fix into master
        git checkout master
        git merge fix
        # look ma no conflict
        ```

        ```mermaid
        graph BT
            classDef master fill:#418a44;
            classDef fix fill:#484A4B;

            subgraph after
                C1a((C1)) --> C0a((C0));
                C2a((C2)) --> C1a;
                C3a((C3)) --> C1a;

                C4a((C4)) --> C3a;
                C4a --> C2a;

                ma("->master<-") --> C4a;
                fa(fix) --> C3a;

                class ma master;
                class fa fix;
            end

            subgraph before
                C1b((C1)) --> C0b((C0));
                C2b((C2)) --> C1b;
                C3b((C3)) --> C1b;

                mb("->master<-") --> C2b;
                fb(fix) --> C3b;

                class mb master
                class fb fix
            end
        ```

    2. Rebase
        Copy the history from the branch point to another point

        ```bash
        # Rebase fix on top of master
        git checkout fix
        git rebase master
        ```

        ```mermaid
        graph BT
            classDef dashed stroke-dasharray: 5, 5;
            classDef master fill:#418a44;
            classDef fix fill:#484A4B;

            subgraph after
                C1a((C1)) --> C0a((C0));
                C2a((C2)) --> C1a;
                C3a((C3)) -.-> C1a;
                C4a((C4)) -.-> C3a;

                C3a_((C3')) --> C2a;
                C4a_((C4')) --> C3a_;

                ma(master) --> C2a;
                fa("->fix<-") --> C4a_;

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
                fb("->fix<-") --> C4b

                class mb master;
                class fb fix;
            end
        ```

    3. Merge vs Rebase
        Merge: preserve history. Rebase: create new commits (replay).
        Conflict is inevitable. Resolving is down to us.

2. Move commit around
    1. Cherry-pick
        Manually copy commits then add it over `HEAD`

        ```bash
        git cherry-pick commit_1 commit_2 ...
        ```

    2. Interactive rebase

## Git Internal

1. How git stores data (or what a commit actually is)
    - Git stores snapshot, not diff
        - The main difference between Git & other VCS
        - Hidden cause: distributed vs centralized
    - Snapshot is large, how to minimize?
        - Refer to unchanged file
        - Data-level diff
        - Compress

2. Hashed tree? Sound familiar...
