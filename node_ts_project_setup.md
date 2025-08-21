# Setup a node project
- create a folder for backend ex: api (here)
- go inside api using below
```
cd api
```
- Now use below command to generate package.json

```
npm init -y
```
### Import required libraries or dependencies

- express
- typescript 
- tsx 
- @types/express
- many more ....

```cmd
npm install express

```
- here -D flag add these into dev dependencies
```cmd
npm install -D typescript tsx @types/express
```

### Create tsconfig.json file
- use below command 

```
npx tsc --init
```


- below kind of tsconfig file will be generated 

```json

{
  // Visit https://aka.ms/tsconfig to read more about this file
  "compilerOptions": {
    // File Layout
    // "rootDir": "./src",
    // "outDir": "./dist",

    // Environment Settings
    // See also https://aka.ms/tsconfig/module
    "module": "nodenext",
    "target": "esnext",
    "types": [],
    // For nodejs:
    // "lib": ["esnext"],
    // "types": ["node"],
    // and npm install -D @types/node

    // Other Outputs
    "sourceMap": true,
    "declaration": true,
    "declarationMap": true,

    // Stricter Typechecking Options
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,

    // Style Options
    // "noImplicitReturns": true,
    // "noImplicitOverride": true,
    // "noUnusedLocals": true,
    // "noUnusedParameters": true,
    // "noFallthroughCasesInSwitch": true,
    // "noPropertyAccessFromIndexSignature": true,

    // Recommended Options
    "strict": true,
    "jsx": "react-jsx",
    "verbatimModuleSyntax": true,
    "isolatedModules": true,
    "noUncheckedSideEffectImports": true,
    "moduleDetection": "force",
    "skipLibCheck": true,
  }
}

```

### Modify package.json Script section

- Add this in script section.
- it will execute ts directly 
- also no need of nodemon watch flag will auto restart server when changes happens
- here index.ts is the actual file from where server is running



```json
  "dev": "tsx watch src/index.ts",
```

- To run our project use 
```
npm run dev
```

- add this build in script 
- it will build project using tsconfig.json file and convert into js 
```json
"build": "tsc",
```

- To build our project use :
```
npm run build
```
## Initalise Git
- to initialise git in project
```git
git init
```

- create a file named `.gitignore `

- add the files and folder names that you downt want to push into github

```
node_modules
.env
dist
```

## Some Useful Git commands

- Clone a remote repository into a local directory.

```
git clone https://github.com/user/repo.git

```

- `git config --global user.name "Your Name"`  :- Set your Git username globally.

```
git config --global user.name "Shubham Kumar"
```

- `git config --global user.email "your@email.com"` — Set your Git email globally.

```
git config --global user.email "shubham@example.com"
```
### Staging & Committing

- `git status` — Show the status of changes (staged, unstaged, untracked).

- `git add <file>` — Stage a single file for commit.
- example:-
```
git add index.js

```

- `git add .` — Stage all changes in the repo.

- `git commit -m "message"` — Commit staged changes with a message.

```
git commit -m "Initial commit"

```

## Branching

- `git branch `— List all branches in the repo.
- `git branch <branch-name>` — Create a new local branch.

```
git branch feature-x

```
- `git checkout <branch-name>` — Switch to another branch.

- `git checkout -b <branch-name> `— Create and switch to a new branch.

```
git checkout -b feature-y

```
- `git branch -d <branch-name> `— Delete a local branch.
```
git branch -d feature-y

```

## Remote Repositories

- `git fetch origin`
- `git branch -r`


- `git remote -v` — Show configured remote repositories.

- `git checkout -b <local-branch> origin/<remote-branch>` — Create a local branch that tracks a remote branch.

```
git checkout -b feature-x origin/feature-x

```

- `git switch <remote-branch> `— (Git 2.23+) Switch to a remote branch and create local branch automatically.

- git pull fetches changes from a remote repository and merges them into your current local branch. When pulling from a specific branch, you explicitly tell Git which remote branch to fetch and merge.
```
git pull <remote> <remote-branch>:<local-branch>
```



- `git pull` — Fetch changes from the remote repository and merge them into your current branch.  
  ```bash
  git pull origin main
  ```
- origin → name of the remote repository (default when you clone).

- main → remote branch you want to pull from.

- Merges the changes from origin/main into your currently checked out branch.

- `git pull origin feature-x:dev-feature-x`:- Pull remote feature-x into local dev-feature-x

- `git pull --rebase origin main`:- Pull and rebase instead of merge (keeps history cleaner):
- Instead of merging, your local commits are re-applied on top of the remote branch, avoiding unnecessary merge commits.

- Recommended when collaborating in teams for a linear history.

```
git pull --rebase origin main

```


## Undo & Fix

- `git log` — View commit history.
- `git diff` — Show unstaged changes in files.
- `git diff --staged` — Show staged changes.

- `git reset --hard `— Reset repo to last commit, discarding all changes.
- `git revert <commit-hash> `— Create a new commit that undoes a past commit.

```
git revert a1b2c3d

```

