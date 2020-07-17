---
title: "nvim-typescript"
date: 2018-05-30T20:35:22-04:00
thumbnail: "/img/nvim-typescript/header.png"
draft: false
---

These days it seems like everyone is using VSCode as their main editor, and with good reasons. VScode offers a lot of cool features, and if you're using TypeScript, you get great support out of the box. But, I'm a vim person, and more specifically, I use [Neovim](https://neovim.io). Since, I write mostly TypeScript, I needed something that offered all the feature of VSCode, but for Noevim. So I created [nvim-typescript](https://github.com/mhartington/nvim-typescript).

### Scratching an itch

Having used vim for quite some time, I was wanting to write a plugin to learn some more about the editor. But after looking into Vimscript, I really didn't quite get it. Instead, I went down the path using Neovim's remote plugin API.

The remote plugin API allows you to author a Neovim plugin in various languages (Python, Rust, Go for example) and have that communicate back to the main editor through an RPC channel. This makes it possible to write 90% of a plugin without having to touch Vimscript.

For the first round of working on nvim-typescript, I went ahead and use the [python-client](https://github.com/neovim/python-client) for Neovim. The python-client was one of the first clients authored and had a lot of documentation to reference. For example, a python plugin could be written as:

```python
import neovim

@neovim.plugin
class TestPlugin(object):

    def __init__(self, nvim):
        self.nvim = nvim

    @neovim.function('TestFunction', sync=True)
    def testfunction(self, args):
        return 3

    @neovim.command('TestCommand', nargs='*', range='')
    def testcommand(self, args, range):
        self.nvim.current.line = ('Command with args: {}, range: {}'
                                  .format(args, range))

    @neovim.autocmd('BufEnter', pattern='*.py', eval='expand("<afile>")', sync=True)
    def on_bufenter(self, filename):
        self.nvim.out_write('testplugin is in ' + filename + '\n')
```

For python folks, this should feel really familiar, and for vim folks, this should feel pretty familiar as well.

After playing with the API for a bit, I authored the first version of nvim-typescript. The plugin worked really well, even if it was very minimal at first. Thankfully, there was some interest in the plugin and some contributors came to add new features and improve the code base. Thanks to all those who contributed! The code base is as much yours as it is mine.


### Moving to Node

After few new release of TypeScript, it was becoming clear that python had some limitations.

- Subprocesses could easily get blocked if the amount of data was too high
- Subprocesses also ran into memory issues when executing long running tasks
- Managing stdout in python felt limited and awkward

I wanted to use JavaScript, but the node-client for Neovim was in rough shape. Thankfully, [billyvg](https://twitter.com/billyvg) was interested in the node-client and offered to pick up the project. After billy got the initial work going, I submitted a PR to rewrite the client again, this time using TypeScript. After some testing, the node-client was ready for use! Billy was also able to add a proper node-provider to Neovim's core. Meaning that node-based plugins were now officially a first class citizen in Neovim.

After all this, I found the time to rewrite the entire python code base, this time in TypeScript, using the node-client API. It's a bit funny...I used a TypeScript project to test and author a TypeScript plugin. Fun times!

### Using the plugin

Now that you know some of the back story, let's actually look at the plugin and it's features.


First, you'll want to install the plugin. You can use any plugin manager you'd like, but I tend to prefer [dein.vim](https://github.com/Shougo/dein.vim) by Shougo.

```viml
call dein#add('mhartington/nvim-typescript', {'build': './install.sh'})
```

You also need a global install of the neovim client.

```shell
npm install -g neovim
```

Optionally you could install [Deoplete.nvim](https://github.com/Shougo/deoplete.nvim) and [Denite](https://github.com/Shougo/denite.nvim) for asynchronous completion and search interfaces.

After this is setup, open Neovim and run `:UpdateRemotePlugins`. If all goes well, you should see

```
remote/host: node host registered plugins ['nvim_typescript']
remote/host: python3 host registered plugins ['denite' 'deoplete']
remote/host: generated rplugin manifest: /Users/mhartington/.local/share/nvim/rplugin.vim
```

Printed out to the console.

### Features

The plugin has a bunch of commands, all documented in the main help file. The commands focus around the idea of `TS<Action>`.


For example, if I wanted to get the `type` of a symbol, I'd hover over the symbol and run `TSType`
![](/img/nvim-typescript/ts-type.gif)

Or if I wanted to see a symbols definition, I'd run `TSDef`. Optionally I could run `TSDefPreview` and open the definition in a split window (think VSCodes "Peek Definition")


![](/img/nvim-typescript/ts-def.gif)

Or, if I wanted to edit the project tsconfig file, I could run `TSEditConfig`. After saving the file, the project and server instance would reload to read the updated settings.

![](/img/nvim-typescript/ts-edit-config.gif)


Now this last piece is more fun than functional I think...but you can also search your entire project for a specific symbol. This does require Denite, but I think it's worth it.


![](/img/nvim-typescript/ts-workspace.gif)


There's also the ability to rename symbols, fix code imports, look up documentation, and get code completion.

### Try it out

Please, feel free to try the plugin out. If you've wanted to use Neovim, but also wanted some great TypeScript features, try nvim-typescript out. And if you do already use the plugin, thanks! Let me know what features you would like to see added!
