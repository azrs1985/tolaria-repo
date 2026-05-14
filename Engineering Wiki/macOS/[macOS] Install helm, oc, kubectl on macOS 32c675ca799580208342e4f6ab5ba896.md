# [macOS] Install helm, oc, kubectl on macOS

Owner: Nam Tran
Last edited time: March 23, 2026 5:55 PM

Install `helm`, `oc`, `kubectl` on macOS:

```jsx
brew install helm kubectl openshift-cli
```

Make sure you have zsh-completions installed

```jsx
brew install zsh-completions
```

Add this to your ~/.zshrc (before the source <(kubectl completion zsh) line):

```jsx
autoload -Uz compinit
compinit
```

Most modern macOS versions use **zsh** by default. Run:

```jsx
echo $SHELL
```

### Autocomplete for kubectl

```jsx
source <(kubectl completion zsh)
```

To make it permanent, add this line to your `~/.zshrc`:

```jsx
echo "source <(kubectl completion zsh)" >> ~/.zshrc
```

### Autocomplete for oc

OpenShift CLI (`oc`) uses the same mechanism as `kubectl`:

```jsx
source <(oc completion zsh)
```

Add permanently:

```jsx
echo "source <(oc completion zsh)" >> ~/.zshrc
```

### Autocomplete for Helm

```jsx
source <(helm completion zsh)
```

Add permanently:

```jsx
echo "source <(helm completion zsh)" >> ~/.zshrc
```

After editing your shell config file (`~/.zshrc`), reload it:

```jsx
source ~/.zshr
```