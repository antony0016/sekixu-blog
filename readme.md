# SekiXu dev note

## Install

install hugo theme

```bash
git submodule add https://github.com/CaiJimmy/hugo-theme-stack/ themes/hugo-theme-stack
```

- update theme config >> config.yaml
- find params > mainSections and change it to `mainSections: ["posts"]`

## Run

```bash
# run hugo server with draft
hugo server -D

# run hugo server without draft
hugo server
```
