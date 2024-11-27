> This action originates from [shirakiya/readme-tree-writer](https://github.com/shirakiya/readme-tree-writer). The only difference is that this action includes all files except .git folder. See [this repos command execution](https://github.com/trueberryless-org/readme-tree-writer?tab=readme-ov-file#tree-and-its-outputs) vs [shirakiya's repos command execution](https://github.com/shirakiya/readme-tree-writer?tab=readme-ov-file#tree-and-its-outputs).

# readme-tree-writer-all

GitHub Action to write the output of "tree" command to each README in your project.

Here is a [sample repository using the original action](https://github.com/shirakiya/readme-tree-writer-sample).

## Usage

Apply to `uses` in workflow config like below.

```yaml
- uses: trueberryless-org/readme-tree-writer-all@v2
```

### Example workflow

This action is helpful to check for maintenance that there are any diff between written tree content in README and actual.
Following workflow is an example to check the diff and fail workflow if the diff is exist.

```yaml
name: Check tree diff

on: push

jobs:
  sample-workflow:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Write tree outputs to README.md
        uses: trueberryless-org/readme-tree-writer-all@v2
        with:
          config_path: .github/readmetreerc.yml
      - name: Check diff
        run: |
          if [ "$(git diff --ignore-space-at-eol . | wc -l)" -gt "0" ]; then
            echo "Detected diff.  See status below:"
            git --no-pager diff HEAD .
            git status
            exit 1
          fi
```

### Action Inputs

| input       | description                                                                          |
| ----------- | ------------------------------------------------------------------------------------ |
| config_path | The path of configuration file for this action (default: `.github/readmetreerc.yml`) |

### Action Outputs

This action does not have any outputs.

### Configuration

This action can be configured by YAML formatted file. It refers to `.github/readmetreerc.yml` in defualt, but you can specify with input parameters `config_path`.  
The parameters that can be set are listed below.

| parameter   | description                                                                                                                                           |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `fileNames` | The list of file names that are scanned. (default: `README.md`)                                                                                       |
| `chapter`   | The chapter name where this action writes tree output. Lines that starts with `#` are regarded (In other words `#`, `##`, `###` and so are included). |
| `include`   | The list of directories that are scanned whether files named `fileNames` param are exist. (default: `.`)                                              |
| `exclude`   | The list of directories that are not scanned. (no default)                                                                                            |

Here is an example.

```yaml
fileNames:
  - README.md
  - README_dev.md
chapter: Tree
include:
  - "."
exclude:
  - node_modules
```

## Details

### Feature

This action writes the results of the `tree` command **in each directory where the README exists** to each README.

For example, when the directory structure is as follows, the following is written in each README.

```
.
├── dir1
│   ├── README.md
│   ├── dir1
│   │   └── file
│   └── dir2
│       └── file
└── dir2
    ├── README.md
    └── file
```

- `./dir1/README.md`

````
# Tree

```
.
├── README.md
├── dir1
│   └── file
└── dir2
    └── file

```

````

- `./dir2/README.md`

````
# Tree

```
.
├── README.md
└── file
```

````

### tree and its outputs

This action uses following command and options in each directory to get tree output.

```
tree --noreport -v -a -I .git .
```

And writes the output to README file. You should pay attension to new lines.

````
# Chapter

```
.
├── dir1
│   ├── file
│   └── dir1-1
│       └── file
└── dir2
    └── file

```

````

## Usecase

Show example workflow settings for some usecase.

### Add a commit if differences by this action apprear

```yaml
name: Add tree changes

on: push

jobs:
  commit-tree-changes:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Write tree outputs to README.md
        uses: trueberryless-org/readme-tree-writer-all@v2
        with:
          config_path: .github/readmetreerc.yml
      - name: Commit if diff exists
        run: |
          if [ "$(git diff --ignore-space-at-eol . | wc -l)" -gt "0" ]; then
            git config user.name github-actions
            git config user.email github-actions@github.com
            git add .
            git commit -m "commit by trueberryless-org/readme-tree-writer-all"
            git push
          fi
```

## License

Licensed under the MIT license, Copyright © trueberryless.

See [LICENSE](/LICENSE) for more information.
