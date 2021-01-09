---
layout: post
title: "GitHub Codespaces and Swift"
date: 2020-12-03 19:37:22 GMT
categories: swift LSP git
---

Are you waiting for Xcode on your iPad?. Relax, Microsoft is on it. 
GitHub announced [Codespaces](https://github.com/features/codespaces/), a new tool presented as "your instant dev environment". How GitHub is going to give us Xcode on Mobile, thanks to [this](http://lists.llvm.org/pipermail/cfe-dev/2018-April/057668.html)

> We at Apple have decided to switch focus from supporting the libclang-based tooling infrastructure in order to join forces on the Clangd development efforts.

[Clangd](https://clangd.llvm.org) is a Language Server. It helps editors improve the support for different languages. Instead of having to recreate language support for each editor, you can deploy a server that editors can query to have a deeper awareness of the language. The analogy with a Web-based API is more than appropriate because the protocol does resemble HTTP: 

Client and server communicate with each other exchanging messages. The messages are composed of a header and a content part. The content is encoded as JSON. 

Whenever something happens in the editor, it sends a request message to the server, the server then processes the request, and return with an appropriate response. That way, the editor doesn't need to anything about the language. Autocomplete? Sure, no problem, let me ask the Language Server for the name of the vars, and I will show them to you. 

Xcode 11.4 includes `sourcekit-lsp` in its toolchain. LSP stands for Language Server Protocol, and that's good news because it allows third party editors to gain access to a huge corpus of knowledge about ObjC and Swift. For instance, Visual Studio Code can leverage Clang knowledge about Swift without breaking a sweat. 

### Visual Studio Code
As said before, Xcode 11.4 introduced `sourcekit-lsp` We can check that running 

```
❯ xcrun -f sourcekit-lsp
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/sourcekit-lsp
```

Since [VSCode extensions](https://code.visualstudio.com/api/language-extensions/language-server-extension-guide) are written in Javascript, we will Node: 

Although Homebrew do have a node package, we will be using nvm, a Node manager that can swap different versions of NPM, is required. 

To install it, we need to download

```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
```

Let's check nvm installation by checking its version

```
❯ nvm --version
0.32.1
```

Now, we can install the latest LTS (Long-Term Support) with 

```
❯ nvm install --lts
Installing latest LTS version.
VERSION_PATH=''
######################################################################### 100.0%
Computing checksum with shasum -a 256
Checksums matched!
Now using node v14.15.1 (npm v6.14.8)
```  

And make it the active Node release with 

```
❯ nvm use --lts
Now using node v14.15.1 (npm v6.14.8)
```

Now, we need to build and install the SourceKit-LSP Extension for Visual Studio Code. Let's clone the [sourcekit-lsp](https://github.com/apple/sourcekit-lsp) and use NPM to build the extension. 

```
❯ git clone https://github.com/apple/sourcekit-lsp.git
Cloning into 'sourcekit-lsp'...
remote: Enumerating objects: 47, done.
remote: Counting objects: 100% (47/47), done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 6847 (delta 19), reused 22 (delta 11), pack-reused 6800
Receiving objects: 100% (6847/6847), 1.52 MiB | 3.13 MiB/s, done.
Resolving deltas: 100% (5117/5117), done.
❯ cd sourcekit-lsp/Editors/vscode
❯ npm run createDevPackage

> sourcekit-lsp@0.0.1 createDevPackage /Users/arielrodriguez/Documents/sourcekit-lsp/Editors/vscode
> npm install && ./node_modules/.bin/vsce package -o ./out/sourcekit-lsp-vscode-dev.vsix

added 73 packages from 62 contributors and audited 73 packages in 3.693s

2 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities

Executing prepublish script 'npm run vscode:prepublish'...

> sourcekit-lsp@0.0.1 vscode:prepublish /Users/arielrodriguez/Documents/sourcekit-lsp/Editors/vscode
> npm run compile


> sourcekit-lsp@0.0.1 compile /Users/arielrodriguez/Documents/sourcekit-lsp/Editors/vscode
> tsc -p ./

 DONE  Packaged: ./out/sourcekit-lsp-vscode-dev.vsix (104 files, 169.8KB)
 INFO
The latest version of vsce is 1.81.1 and you have 1.79.5.
Update it now: npm install -g vsce
```

And finally, we can install the extension with 

```
❯ code --install-extension out/sourcekit-lsp-vscode-dev.vsix
Installing extensions...
(node:40702) [DEP0005] DeprecationWarning: Buffer() is deprecated due to security and usability issues. Please use the Buffer.alloc(), Buffer.allocUnsafe(), or Buffer.from() methods instead.
Extension 'sourcekit-lsp-vscode-dev.vsix' was successfully installed.
```

Back to the beginning, I've mentioned that GitHub was working to give us Xcode on Mobile. Microsoft is touting Codespaces as "the full Visual Studio Code experience without leaving GitHub" That's why LSP is important. 