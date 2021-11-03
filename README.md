**WIP**

Speed up loading Lua modules in Neovim to improve startup time.

Optimisations include:

* Cache for compiled lua modules.
* Improved default package loader for uncached module loads
* Restoring the preloader

The expectation is that a form of this plugin will eventually be merged into Neovim core via this [PR](https://github.com/neovim/neovim/pull/15436). This plugin serves as a way for impatient users to speed up there Neovim 0.5 until the PR is merged and included in a following Neovim release at which point this plugin will be redundant.

## Optimisations

This plugin does several things to speed up `require` in Lua.

### Implements cache for all loaded Lua modules

This is done by using `loadstring` to compile the Lua modules to bytecode and stores them in a cache file. This also has the benefit of avoiding Neovim's expensive module loader which uses `nvim_get_runtime_file()`. The cache is invalidated using the modified time of each modules file path.

The cache file is located in `$XDG_CACHE_HOME/nvim/luacache`.

### Reduces `runtimepath` during `require`

`runtimepath` contains directories for many things used by Neovim including Lua modules; the full list of what it is used for can be found using `:help 'runtimepath'`. When `require` is called, Neovim searches through every directory in `runtimepath` until it finds a match. This means it ends up searching in every plugin that doesn't have a Lua directory, which can be a lot and makes `require` much more expensive to run. To mitigate this, Impatient reduces `runtimepath` during `require` to only contain directories that have a Lua directory.

### Restores the preloader

Neovim currently places its own loader for searching runtime files at the front of `package.loaders`. This prevents any preloaders in `package.preload` from being used. This plugin fixes that by moving the default package preloader to run before Neovim's loader. For example, LuaJIT provides preloaders for the built-in modules `ffi` and `bit`, so this optimisation will improve the loading of those.

## Installation

[packer.nvim](https://github.com/wbthomason/packer.nvim):
```lua
-- Is using a standard Neovim install, i.e. built from source or using a
-- provided appimage.
use 'lewis6991/impatient.nvim'
```

You may also want Packers generated compiled file to be cached by Impatient too. To enable this:

```lua
packer.startup{{...},
  config = {
    -- Move to lua dir so impatient.nvim can cache it
    compile_path = vim.fn.stdpath('config')..'/lua/packer_compiled.lua'
  }
}

```

Then make sure to add `require('packer_compiled')` somewhere in your config.

## Setup

impatient needs to be setup before any other lua plugin is loaded so it is recommended you add the following near the start of your `init.vim`.

```viml
lua require('impatient')
```

## Commands

`:LuaCacheClear`:

Remove the loaded cache and delete the cache file. A new cache file will be created the next time you load Neovim.

`:LuaCacheLog`:

View log of impatient.

`:LuaCacheProfile`:

View profiling data. To enable, Impatient must be setup with:

```viml
lua require'impatient'.enable_profile()
```

## Performance Example

Measured on a M1 MacBook Air.

<details>
<summary>Standard</summary>

<table>
    <thead>
        <tr><th>Module</th><th>Resolve</th><th>Load</th><th>Total</th></tr>
    </thead>
    <tbody>
        <tr><td>bufferline.buffers</td><td>0.259858ms</td><td>0.087198ms</td><td>0.347056ms</td></tr>
        <tr><td>bufferline.colors</td><td>0.335110ms</td><td>0.508520ms</td><td>0.843630ms</td></tr>
        <tr><td>bufferline.config</td><td>0.259542ms</td><td>0.263600ms</td><td>0.523142ms</td></tr>
        <tr><td>bufferline.constants</td><td>0.290488ms</td><td>0.036354ms</td><td>0.326842ms</td></tr>
        <tr><td>bufferline.custom_area</td><td>0.280831ms</td><td>0.056538ms</td><td>0.337369ms</td></tr>
        <tr><td>bufferline.diagnostics</td><td>0.267217ms</td><td>0.088245ms</td><td>0.355462ms</td></tr>
        <tr><td>bufferline.duplicates</td><td>0.271018ms</td><td>0.073781ms</td><td>0.344799ms</td></tr>
        <tr><td>bufferline.highlights</td><td>0.332502ms</td><td>1.833617ms</td><td>2.166119ms</td></tr>
        <tr><td>bufferline.numbers</td><td>0.275835ms</td><td>0.096760ms</td><td>0.372595ms</td></tr>
        <tr><td>bufferline.offset</td><td>0.269489ms</td><td>0.136608ms</td><td>0.406097ms</td></tr>
        <tr><td>bufferline.pick</td><td>0.266726ms</td><td>0.063065ms</td><td>0.329791ms</td></tr>
        <tr><td>bufferline.sorters</td><td>0.304341ms</td><td>0.090384ms</td><td>0.394725ms</td></tr>
        <tr><td>bufferline.tabs</td><td>0.305597ms</td><td>0.064405ms</td><td>0.370002ms</td></tr>
        <tr><td>bufferline.utils</td><td>0.266954ms</td><td>0.138761ms</td><td>0.405715ms</td></tr>
        <tr><td>bufferline/constants</td><td>0.282626ms</td><td>0.025039ms</td><td>0.307665ms</td></tr>
        <tr><td>bufferline</td><td>0.294972ms</td><td>0.746609ms</td><td>1.041581ms</td></tr>
        <tr><td>cleanfold</td><td>0.265625ms</td><td>0.086065ms</td><td>0.351690ms</td></tr>
        <tr><td>cmp.autocmd</td><td>0.173400ms</td><td>0.032468ms</td><td>0.205868ms</td></tr>
        <tr><td>cmp.config.compare</td><td>0.199557ms</td><td>0.080771ms</td><td>0.280328ms</td></tr>
        <tr><td>cmp.config.default</td><td>0.200404ms</td><td>0.077532ms</td><td>0.277936ms</td></tr>
        <tr><td>cmp.config</td><td>0.186847ms</td><td>0.059070ms</td><td>0.245917ms</td></tr>
        <tr><td>cmp.context</td><td>0.177237ms</td><td>0.097014ms</td><td>0.274251ms</td></tr>
        <tr><td>cmp.core</td><td>0.166483ms</td><td>0.261528ms</td><td>0.428011ms</td></tr>
        <tr><td>cmp.entry</td><td>0.179926ms</td><td>0.284327ms</td><td>0.464253ms</td></tr>
        <tr><td>cmp.float</td><td>0.171692ms</td><td>0.117217ms</td><td>0.288909ms</td></tr>
        <tr><td>cmp.mapping</td><td>0.173689ms</td><td>0.057433ms</td><td>0.231122ms</td></tr>
        <tr><td>cmp.matcher</td><td>0.177659ms</td><td>0.134717ms</td><td>0.312376ms</td></tr>
        <tr><td>cmp.menu</td><td>0.171497ms</td><td>0.158820ms</td><td>0.330317ms</td></tr>
        <tr><td>cmp.source</td><td>0.237351ms</td><td>0.249528ms</td><td>0.486879ms</td></tr>
        <tr><td>cmp.types.cmp</td><td>0.171394ms</td><td>0.026495ms</td><td>0.197889ms</td></tr>
        <tr><td>cmp.types.lsp</td><td>0.170047ms</td><td>0.088564ms</td><td>0.258611ms</td></tr>
        <tr><td>cmp.types.vim</td><td>0.180930ms</td><td>0.011885ms</td><td>0.192815ms</td></tr>
        <tr><td>cmp.types</td><td>0.809259ms</td><td>0.016533ms</td><td>0.825792ms</td></tr>
        <tr><td>cmp.utils.async</td><td>0.178425ms</td><td>0.054211ms</td><td>0.232636ms</td></tr>
        <tr><td>cmp.utils.cache</td><td>0.178923ms</td><td>0.036663ms</td><td>0.215586ms</td></tr>
        <tr><td>cmp.utils.char</td><td>0.188165ms</td><td>0.075577ms</td><td>0.263742ms</td></tr>
        <tr><td>cmp.utils.check</td><td>0.182193ms</td><td>0.028363ms</td><td>0.210556ms</td></tr>
        <tr><td>cmp.utils.debug</td><td>0.184229ms</td><td>0.024848ms</td><td>0.209077ms</td></tr>
        <tr><td>cmp.utils.keymap</td><td>0.180097ms</td><td>0.142296ms</td><td>0.322393ms</td></tr>
        <tr><td>cmp.utils.misc</td><td>0.176499ms</td><td>0.094667ms</td><td>0.271166ms</td></tr>
        <tr><td>cmp.utils.pattern</td><td>0.188940ms</td><td>0.022397ms</td><td>0.211337ms</td></tr>
        <tr><td>cmp.utils.str</td><td>0.175312ms</td><td>0.110933ms</td><td>0.286245ms</td></tr>
        <tr><td>cmp_buffer.buffer</td><td>0.281058ms</td><td>0.114087ms</td><td>0.395145ms</td></tr>
        <tr><td>cmp_buffer</td><td>0.936429ms</td><td>0.108692ms</td><td>1.045121ms</td></tr>
        <tr><td>cmp_luasnip</td><td>0.967596ms</td><td>0.123955ms</td><td>1.091551ms</td></tr>
        <tr><td>cmp_nvim_lsp.source</td><td>0.254415ms</td><td>0.097778ms</td><td>0.352193ms</td></tr>
        <tr><td>cmp_nvim_lsp</td><td>0.950312ms</td><td>0.078823ms</td><td>1.029135ms</td></tr>
        <tr><td>cmp_nvim_lua</td><td>0.929715ms</td><td>0.111288ms</td><td>1.041003ms</td></tr>
        <tr><td>cmp_path</td><td>0.903813ms</td><td>0.163515ms</td><td>1.067328ms</td></tr>
        <tr><td>cmp</td><td>0.776393ms</td><td>0.101589ms</td><td>0.877982ms</td></tr>
        <tr><td>colorizer/nvim</td><td>0.184606ms</td><td>0.156782ms</td><td>0.341388ms</td></tr>
        <tr><td>colorizer/trie</td><td>0.175391ms</td><td>0.162284ms</td><td>0.337675ms</td></tr>
        <tr><td>colorizer</td><td>0.179935ms</td><td>0.498320ms</td><td>0.678255ms</td></tr>
        <tr><td>compe_tmux.source</td><td>0.244179ms</td><td>0.066047ms</td><td>0.310226ms</td></tr>
        <tr><td>compe_tmux.tmux</td><td>0.230539ms</td><td>0.077219ms</td><td>0.307758ms</td></tr>
        <tr><td>compe_tmux.utils</td><td>0.230812ms</td><td>0.032568ms</td><td>0.263380ms</td></tr>
        <tr><td>compe_tmux</td><td>0.946990ms</td><td>0.024452ms</td><td>0.971442ms</td></tr>
        <tr><td>foldsigns</td><td>0.238926ms</td><td>0.098872ms</td><td>0.337798ms</td></tr>
        <tr><td>fzf_lib</td><td>0.113888ms</td><td>0.065426ms</td><td>0.179314ms</td></tr>
        <tr><td>gitsigns.cache</td><td>0.213791ms</td><td>0.715639ms</td><td>0.929430ms</td></tr>
        <tr><td>gitsigns.config</td><td>0.216884ms</td><td>0.789166ms</td><td>1.006050ms</td></tr>
        <tr><td>gitsigns.current_line_blame</td><td>0.262493ms</td><td>0.506303ms</td><td>0.768796ms</td></tr>
        <tr><td>gitsigns.debounce</td><td>0.213720ms</td><td>0.635708ms</td><td>0.849428ms</td></tr>
        <tr><td>gitsigns.debug</td><td>0.216191ms</td><td>0.594014ms</td><td>0.810205ms</td></tr>
        <tr><td>gitsigns.git</td><td>0.207405ms</td><td>0.761158ms</td><td>0.968563ms</td></tr>
        <tr><td>gitsigns.highlight</td><td>0.221700ms</td><td>0.686345ms</td><td>0.908045ms</td></tr>
        <tr><td>gitsigns.hunks</td><td>0.215175ms</td><td>0.727926ms</td><td>0.943101ms</td></tr>
        <tr><td>gitsigns.manager</td><td>0.220365ms</td><td>0.615187ms</td><td>0.835552ms</td></tr>
        <tr><td>gitsigns.signs</td><td>0.211247ms</td><td>0.471879ms</td><td>0.683126ms</td></tr>
        <tr><td>gitsigns.status</td><td>0.221054ms</td><td>0.656612ms</td><td>0.877666ms</td></tr>
        <tr><td>gitsigns.util</td><td>0.217103ms</td><td>0.661626ms</td><td>0.878729ms</td></tr>
        <tr><td>gitsigns</td><td>0.206139ms</td><td>0.757668ms</td><td>0.963807ms</td></tr>
        <tr><td>lspconfig.ui.windows</td><td>0.196567ms</td><td>0.113610ms</td><td>0.310177ms</td></tr>
        <tr><td>lspconfig/configs</td><td>0.188288ms</td><td>0.185246ms</td><td>0.373534ms</td></tr>
        <tr><td>lspconfig/util</td><td>0.196135ms</td><td>0.250408ms</td><td>0.446543ms</td></tr>
        <tr><td>lspconfig</td><td>0.181835ms</td><td>0.085778ms</td><td>0.267613ms</td></tr>
        <tr><td>lspinstall/servers/angular</td><td>0.184711ms</td><td>0.017180ms</td><td>0.201891ms</td></tr>
        <tr><td>lspinstall/servers/bash</td><td>0.262689ms</td><td>0.035765ms</td><td>0.298454ms</td></tr>
        <tr><td>lspinstall/servers/clojure</td><td>0.193567ms</td><td>0.019951ms</td><td>0.213518ms</td></tr>
        <tr><td>lspinstall/servers/cmake</td><td>0.180468ms</td><td>0.015740ms</td><td>0.196208ms</td></tr>
        <tr><td>lspinstall/servers/cpp</td><td>0.185474ms</td><td>0.017886ms</td><td>0.203360ms</td></tr>
        <tr><td>lspinstall/servers/csharp</td><td>0.194527ms</td><td>0.039318ms</td><td>0.233845ms</td></tr>
        <tr><td>lspinstall/servers/css</td><td>0.197305ms</td><td>0.024359ms</td><td>0.221664ms</td></tr>
        <tr><td>lspinstall/servers/deno</td><td>0.199388ms</td><td>0.025577ms</td><td>0.224965ms</td></tr>
        <tr><td>lspinstall/servers/diagnosticls</td><td>0.208307ms</td><td>0.018917ms</td><td>0.227224ms</td></tr>
        <tr><td>lspinstall/servers/dockerfile</td><td>0.189822ms</td><td>0.015957ms</td><td>0.205779ms</td></tr>
        <tr><td>lspinstall/servers/efm</td><td>0.184480ms</td><td>0.016019ms</td><td>0.200499ms</td></tr>
        <tr><td>lspinstall/servers/elixir</td><td>0.189127ms</td><td>0.016280ms</td><td>0.205407ms</td></tr>
        <tr><td>lspinstall/servers/elm</td><td>0.186706ms</td><td>0.019313ms</td><td>0.206019ms</td></tr>
        <tr><td>lspinstall/servers/ember</td><td>0.189023ms</td><td>0.017318ms</td><td>0.206341ms</td></tr>
        <tr><td>lspinstall/servers/fortran</td><td>0.191423ms</td><td>0.015638ms</td><td>0.207061ms</td></tr>
        <tr><td>lspinstall/servers/go</td><td>0.173845ms</td><td>0.015202ms</td><td>0.189047ms</td></tr>
        <tr><td>lspinstall/servers/graphql</td><td>0.182745ms</td><td>0.022024ms</td><td>0.204769ms</td></tr>
        <tr><td>lspinstall/servers/haskell</td><td>0.185940ms</td><td>0.016817ms</td><td>0.202757ms</td></tr>
        <tr><td>lspinstall/servers/html</td><td>0.188460ms</td><td>0.030898ms</td><td>0.219358ms</td></tr>
        <tr><td>lspinstall/servers/java</td><td>0.175670ms</td><td>0.075387ms</td><td>0.251057ms</td></tr>
        <tr><td>lspinstall/servers/json</td><td>0.176454ms</td><td>0.020543ms</td><td>0.196997ms</td></tr>
        <tr><td>lspinstall/servers/kotlin</td><td>0.187049ms</td><td>0.016436ms</td><td>0.203485ms</td></tr>
        <tr><td>lspinstall/servers/latex</td><td>0.187523ms</td><td>0.017197ms</td><td>0.204720ms</td></tr>
        <tr><td>lspinstall/servers/lua</td><td>0.177295ms</td><td>0.019663ms</td><td>0.196958ms</td></tr>
        <tr><td>lspinstall/servers/php</td><td>0.174703ms</td><td>0.022575ms</td><td>0.197278ms</td></tr>
        <tr><td>lspinstall/servers/puppet</td><td>0.178470ms</td><td>0.014800ms</td><td>0.193270ms</td></tr>
        <tr><td>lspinstall/servers/purescript</td><td>0.186936ms</td><td>0.018114ms</td><td>0.205050ms</td></tr>
        <tr><td>lspinstall/servers/python</td><td>0.180147ms</td><td>0.013925ms</td><td>0.194072ms</td></tr>
        <tr><td>lspinstall/servers/rome</td><td>0.184556ms</td><td>0.017042ms</td><td>0.201598ms</td></tr>
        <tr><td>lspinstall/servers/ruby</td><td>0.200433ms</td><td>0.020969ms</td><td>0.221402ms</td></tr>
        <tr><td>lspinstall/servers/rust</td><td>0.176726ms</td><td>0.018242ms</td><td>0.194968ms</td></tr>
        <tr><td>lspinstall/servers/svelte</td><td>0.178724ms</td><td>0.018540ms</td><td>0.197264ms</td></tr>
        <tr><td>lspinstall/servers/tailwindcss</td><td>0.196462ms</td><td>0.015955ms</td><td>0.212417ms</td></tr>
        <tr><td>lspinstall/servers/terraform</td><td>0.193758ms</td><td>0.018556ms</td><td>0.212314ms</td></tr>
        <tr><td>lspinstall/servers/typescript</td><td>0.184999ms</td><td>0.014601ms</td><td>0.199600ms</td></tr>
        <tr><td>lspinstall/servers/vim</td><td>0.177593ms</td><td>0.014432ms</td><td>0.192025ms</td></tr>
        <tr><td>lspinstall/servers/vue</td><td>0.176739ms</td><td>0.015030ms</td><td>0.191769ms</td></tr>
        <tr><td>lspinstall/servers/yaml</td><td>0.178760ms</td><td>0.018300ms</td><td>0.197060ms</td></tr>
        <tr><td>lspinstall/servers</td><td>0.176806ms</td><td>0.051793ms</td><td>0.228599ms</td></tr>
        <tr><td>lspinstall/util</td><td>0.163294ms</td><td>0.027177ms</td><td>0.190471ms</td></tr>
        <tr><td>lspinstall</td><td>0.179635ms</td><td>0.109075ms</td><td>0.288710ms</td></tr>
        <tr><td>lspkind</td><td>0.853034ms</td><td>0.080067ms</td><td>0.933101ms</td></tr>
        <tr><td>luasnip.config</td><td>0.272452ms</td><td>0.078598ms</td><td>0.351050ms</td></tr>
        <tr><td>luasnip.nodes.choiceNode</td><td>0.290001ms</td><td>0.160706ms</td><td>0.450707ms</td></tr>
        <tr><td>luasnip.nodes.dynamicNode</td><td>0.289569ms</td><td>0.116809ms</td><td>0.406378ms</td></tr>
        <tr><td>luasnip.nodes.functionNode</td><td>0.292459ms</td><td>0.080532ms</td><td>0.372991ms</td></tr>
        <tr><td>luasnip.nodes.insertNode</td><td>0.290410ms</td><td>0.124346ms</td><td>0.414756ms</td></tr>
        <tr><td>luasnip.nodes.node</td><td>0.286651ms</td><td>0.103704ms</td><td>0.390355ms</td></tr>
        <tr><td>luasnip.nodes.snippet</td><td>0.285687ms</td><td>0.473502ms</td><td>0.759189ms</td></tr>
        <tr><td>luasnip.nodes.textNode</td><td>0.279409ms</td><td>0.035724ms</td><td>0.315133ms</td></tr>
        <tr><td>luasnip.session</td><td>0.281463ms</td><td>0.011938ms</td><td>0.293401ms</td></tr>
        <tr><td>luasnip.util.environ</td><td>0.277413ms</td><td>0.276535ms</td><td>0.553948ms</td></tr>
        <tr><td>luasnip.util.events</td><td>0.288992ms</td><td>0.025592ms</td><td>0.314584ms</td></tr>
        <tr><td>luasnip.util.functions</td><td>0.282744ms</td><td>0.020942ms</td><td>0.303686ms</td></tr>
        <tr><td>luasnip.util.mark</td><td>0.269638ms</td><td>0.104326ms</td><td>0.373964ms</td></tr>
        <tr><td>luasnip.util.parser</td><td>0.283692ms</td><td>0.267040ms</td><td>0.550732ms</td></tr>
        <tr><td>luasnip.util.pattern_tokenizer</td><td>0.346878ms</td><td>0.075790ms</td><td>0.422668ms</td></tr>
        <tr><td>luasnip.util.types</td><td>0.272280ms</td><td>0.022731ms</td><td>0.295011ms</td></tr>
        <tr><td>luasnip.util.util</td><td>0.268698ms</td><td>0.351236ms</td><td>0.619934ms</td></tr>
        <tr><td>luasnip</td><td>0.974144ms</td><td>0.281725ms</td><td>1.255869ms</td></tr>
        <tr><td>null-ls.builtins.code-actions</td><td>0.222739ms</td><td>0.055434ms</td><td>0.278173ms</td></tr>
        <tr><td>null-ls.builtins.diagnostics</td><td>0.202345ms</td><td>0.508787ms</td><td>0.711132ms</td></tr>
        <tr><td>null-ls.builtins.formatting</td><td>0.226301ms</td><td>0.399759ms</td><td>0.626060ms</td></tr>
        <tr><td>null-ls.builtins.test</td><td>0.191547ms</td><td>0.144765ms</td><td>0.336312ms</td></tr>
        <tr><td>null-ls.builtins</td><td>1.004278ms</td><td>0.045466ms</td><td>1.049744ms</td></tr>
        <tr><td>null-ls.code-actions</td><td>0.191288ms</td><td>0.067136ms</td><td>0.258424ms</td></tr>
        <tr><td>null-ls.config</td><td>0.251753ms</td><td>0.191202ms</td><td>0.442955ms</td></tr>
        <tr><td>null-ls.diagnostics</td><td>0.188810ms</td><td>0.118083ms</td><td>0.306893ms</td></tr>
        <tr><td>null-ls.formatting</td><td>0.186930ms</td><td>0.132342ms</td><td>0.319272ms</td></tr>
        <tr><td>null-ls.generators</td><td>0.191108ms</td><td>0.142183ms</td><td>0.333291ms</td></tr>
        <tr><td>null-ls.handlers</td><td>0.184452ms</td><td>0.061545ms</td><td>0.245997ms</td></tr>
        <tr><td>null-ls.helpers</td><td>0.319042ms</td><td>1.003023ms</td><td>1.322065ms</td></tr>
        <tr><td>null-ls.info</td><td>0.197606ms</td><td>0.091835ms</td><td>0.289441ms</td></tr>
        <tr><td>null-ls.logger</td><td>0.186859ms</td><td>0.025955ms</td><td>0.212814ms</td></tr>
        <tr><td>null-ls.loop</td><td>0.187206ms</td><td>0.189593ms</td><td>0.376799ms</td></tr>
        <tr><td>null-ls.lspconfig</td><td>0.210560ms</td><td>0.101977ms</td><td>0.312537ms</td></tr>
        <tr><td>null-ls.methods</td><td>0.317264ms</td><td>0.096905ms</td><td>0.414169ms</td></tr>
        <tr><td>null-ls.rpc</td><td>0.178238ms</td><td>0.136415ms</td><td>0.314653ms</td></tr>
        <tr><td>null-ls.state</td><td>0.179769ms</td><td>0.077196ms</td><td>0.256965ms</td></tr>
        <tr><td>null-ls.utils</td><td>0.361939ms</td><td>0.309648ms</td><td>0.671587ms</td></tr>
        <tr><td>null-ls</td><td>0.878750ms</td><td>0.072096ms</td><td>0.950846ms</td></tr>
        <tr><td>nvim-treesitter-playground</td><td>0.198691ms</td><td>0.053470ms</td><td>0.252161ms</td></tr>
        <tr><td>nvim-treesitter.caching</td><td>0.200836ms</td><td>0.066041ms</td><td>0.266877ms</td></tr>
        <tr><td>nvim-treesitter.configs</td><td>0.175211ms</td><td>0.352484ms</td><td>0.527695ms</td></tr>
        <tr><td>nvim-treesitter.info</td><td>0.183248ms</td><td>0.181651ms</td><td>0.364899ms</td></tr>
        <tr><td>nvim-treesitter.install</td><td>0.177862ms</td><td>0.528773ms</td><td>0.706635ms</td></tr>
        <tr><td>nvim-treesitter.parsers</td><td>0.172642ms</td><td>0.504748ms</td><td>0.677390ms</td></tr>
        <tr><td>nvim-treesitter.query_predicates</td><td>0.206420ms</td><td>0.089659ms</td><td>0.296079ms</td></tr>
        <tr><td>nvim-treesitter.query</td><td>0.180591ms</td><td>0.276378ms</td><td>0.456969ms</td></tr>
        <tr><td>nvim-treesitter.shell_command_selectors</td><td>0.208011ms</td><td>0.390012ms</td><td>0.598023ms</td></tr>
        <tr><td>nvim-treesitter.ts_utils</td><td>0.196365ms</td><td>0.578957ms</td><td>0.775322ms</td></tr>
        <tr><td>nvim-treesitter.tsrange</td><td>0.176846ms</td><td>0.281337ms</td><td>0.458183ms</td></tr>
        <tr><td>nvim-treesitter.utils</td><td>0.190822ms</td><td>0.138708ms</td><td>0.329530ms</td></tr>
        <tr><td>nvim-treesitter</td><td>0.179197ms</td><td>0.084622ms</td><td>0.263819ms</td></tr>
        <tr><td>nvim-web-devicons</td><td>0.161349ms</td><td>0.423220ms</td><td>0.584569ms</td></tr>
        <tr><td>octo.base64</td><td>0.073452ms</td><td>0.166600ms</td><td>0.240052ms</td></tr>
        <tr><td>octo.colors</td><td>0.072182ms</td><td>0.220893ms</td><td>0.293075ms</td></tr>
        <tr><td>octo.completion</td><td>0.042705ms</td><td>0.076242ms</td><td>0.118947ms</td></tr>
        <tr><td>octo.config</td><td>0.040683ms</td><td>0.103420ms</td><td>0.144103ms</td></tr>
        <tr><td>octo.constants</td><td>0.048190ms</td><td>0.036868ms</td><td>0.085058ms</td></tr>
        <tr><td>octo.date</td><td>0.040060ms</td><td>0.942421ms</td><td>0.982481ms</td></tr>
        <tr><td>octo.folds</td><td>0.040495ms</td><td>0.025282ms</td><td>0.065777ms</td></tr>
        <tr><td>octo.gh</td><td>0.041307ms</td><td>0.082782ms</td><td>0.124089ms</td></tr>
        <tr><td>octo.graphql</td><td>0.041525ms</td><td>0.284976ms</td><td>0.326501ms</td></tr>
        <tr><td>octo.mappings</td><td>0.046533ms</td><td>0.146523ms</td><td>0.193056ms</td></tr>
        <tr><td>octo.model.body-metadata</td><td>0.050118ms</td><td>0.027039ms</td><td>0.077157ms</td></tr>
        <tr><td>octo.model.octo-buffer</td><td>0.050438ms</td><td>0.510193ms</td><td>0.560631ms</td></tr>
        <tr><td>octo.model.thread-metadata</td><td>0.051801ms</td><td>0.021644ms</td><td>0.073445ms</td></tr>
        <tr><td>octo.model.title-metadata</td><td>0.044060ms</td><td>0.019239ms</td><td>0.063299ms</td></tr>
        <tr><td>octo.reviews.file-entry</td><td>0.048714ms</td><td>0.297213ms</td><td>0.345927ms</td></tr>
        <tr><td>octo.reviews.file-panel</td><td>0.046898ms</td><td>0.332794ms</td><td>0.379692ms</td></tr>
        <tr><td>octo.reviews.layout</td><td>0.049106ms</td><td>0.190465ms</td><td>0.239571ms</td></tr>
        <tr><td>octo.reviews.renderer</td><td>0.045927ms</td><td>0.073972ms</td><td>0.119899ms</td></tr>
        <tr><td>octo.reviews</td><td>0.043867ms</td><td>0.420502ms</td><td>0.464369ms</td></tr>
        <tr><td>octo.signs</td><td>0.038892ms</td><td>0.066371ms</td><td>0.105263ms</td></tr>
        <tr><td>octo.ui.bubbles</td><td>0.040408ms</td><td>0.066573ms</td><td>0.106981ms</td></tr>
        <tr><td>octo.utils</td><td>0.039183ms</td><td>0.759759ms</td><td>0.798942ms</td></tr>
        <tr><td>octo.window</td><td>0.039895ms</td><td>0.177679ms</td><td>0.217574ms</td></tr>
        <tr><td>octo.writers</td><td>0.040878ms</td><td>0.983512ms</td><td>1.024390ms</td></tr>
        <tr><td>octo</td><td>0.044227ms</td><td>0.245518ms</td><td>0.289745ms</td></tr>
        <tr><td>packer.load</td><td>0.236989ms</td><td>0.179790ms</td><td>0.416779ms</td></tr>
        <tr><td>plenary.async.async</td><td>0.161200ms</td><td>0.081868ms</td><td>0.243068ms</td></tr>
        <tr><td>plenary.async.control</td><td>0.152830ms</td><td>0.124154ms</td><td>0.276984ms</td></tr>
        <tr><td>plenary.async.structs</td><td>0.178025ms</td><td>0.066626ms</td><td>0.244651ms</td></tr>
        <tr><td>plenary.async.util</td><td>0.143275ms</td><td>0.090109ms</td><td>0.233384ms</td></tr>
        <tr><td>plenary.async_lib.api</td><td>0.145936ms</td><td>0.025156ms</td><td>0.171092ms</td></tr>
        <tr><td>plenary.async_lib.async</td><td>0.148677ms</td><td>0.193933ms</td><td>0.342610ms</td></tr>
        <tr><td>plenary.async_lib.lsp</td><td>0.150125ms</td><td>0.022988ms</td><td>0.173113ms</td></tr>
        <tr><td>plenary.async_lib.structs</td><td>0.167282ms</td><td>0.091045ms</td><td>0.258327ms</td></tr>
        <tr><td>plenary.async_lib.tests</td><td>0.155115ms</td><td>0.027298ms</td><td>0.182413ms</td></tr>
        <tr><td>plenary.async_lib.util</td><td>0.149542ms</td><td>0.249447ms</td><td>0.398989ms</td></tr>
        <tr><td>plenary.async_lib.uv_async</td><td>0.156737ms</td><td>0.054766ms</td><td>0.211503ms</td></tr>
        <tr><td>plenary.async_lib</td><td>0.854841ms</td><td>0.047373ms</td><td>0.902214ms</td></tr>
        <tr><td>plenary.bit</td><td>0.145979ms</td><td>0.226697ms</td><td>0.372676ms</td></tr>
        <tr><td>plenary.errors</td><td>0.138287ms</td><td>0.022672ms</td><td>0.160959ms</td></tr>
        <tr><td>plenary.functional</td><td>0.140045ms</td><td>0.052594ms</td><td>0.192639ms</td></tr>
        <tr><td>plenary.job</td><td>0.142743ms</td><td>0.387862ms</td><td>0.530605ms</td></tr>
        <tr><td>plenary.log</td><td>0.134119ms</td><td>0.147632ms</td><td>0.281751ms</td></tr>
        <tr><td>plenary.path</td><td>0.272007ms</td><td>0.617485ms</td><td>0.889492ms</td></tr>
        <tr><td>plenary.strings</td><td>0.146322ms</td><td>0.126995ms</td><td>0.273317ms</td></tr>
        <tr><td>plenary.tbl</td><td>0.133801ms</td><td>0.042601ms</td><td>0.176402ms</td></tr>
        <tr><td>plenary.vararg.rotate</td><td>0.153851ms</td><td>0.073581ms</td><td>0.227432ms</td></tr>
        <tr><td>plenary.vararg</td><td>0.825993ms</td><td>0.015645ms</td><td>0.841638ms</td></tr>
        <tr><td>spaceless</td><td>0.129994ms</td><td>0.085178ms</td><td>0.215172ms</td></tr>
        <tr><td>spellsitter</td><td>0.464111ms</td><td>0.981334ms</td><td>1.445445ms</td></tr>
        <tr><td>symbols-outline.config</td><td>0.130065ms</td><td>0.104856ms</td><td>0.234921ms</td></tr>
        <tr><td>symbols-outline.markdown</td><td>0.139292ms</td><td>0.037662ms</td><td>0.176954ms</td></tr>
        <tr><td>symbols-outline.parser</td><td>0.141076ms</td><td>0.150265ms</td><td>0.291341ms</td></tr>
        <tr><td>symbols-outline.symbols</td><td>0.136039ms</td><td>0.031510ms</td><td>0.167549ms</td></tr>
        <tr><td>symbols-outline.ui</td><td>0.128141ms</td><td>0.077715ms</td><td>0.205856ms</td></tr>
        <tr><td>symbols-outline.utils.init</td><td>0.144694ms</td><td>0.033804ms</td><td>0.178498ms</td></tr>
        <tr><td>symbols-outline.utils.lsp_utils</td><td>0.154402ms</td><td>0.026658ms</td><td>0.181060ms</td></tr>
        <tr><td>symbols-outline.view</td><td>0.135813ms</td><td>0.041749ms</td><td>0.177562ms</td></tr>
        <tr><td>symbols-outline.writer</td><td>0.132671ms</td><td>0.053070ms</td><td>0.185741ms</td></tr>
        <tr><td>symbols-outline</td><td>0.137599ms</td><td>0.214040ms</td><td>0.351639ms</td></tr>
        <tr><td>telescope._extensions.fzf</td><td>0.174583ms</td><td>0.141541ms</td><td>0.316124ms</td></tr>
        <tr><td>telescope._extensions</td><td>0.802657ms</td><td>0.043015ms</td><td>0.845672ms</td></tr>
        <tr><td>telescope.config</td><td>0.112322ms</td><td>0.223731ms</td><td>0.336053ms</td></tr>
        <tr><td>telescope.deprecated</td><td>0.130086ms</td><td>0.048783ms</td><td>0.178869ms</td></tr>
        <tr><td>telescope.log</td><td>0.109942ms</td><td>0.019130ms</td><td>0.129072ms</td></tr>
        <tr><td>telescope.sorters</td><td>0.125307ms</td><td>0.368071ms</td><td>0.493378ms</td></tr>
        <tr><td>telescope.utils</td><td>0.117569ms</td><td>0.370753ms</td><td>0.488322ms</td></tr>
        <tr><td>telescope</td><td>1.017321ms</td><td>0.069112ms</td><td>1.086433ms</td></tr>
        <tr><td>treesitter-context.utils</td><td>0.183435ms</td><td>0.046763ms</td><td>0.230198ms</td></tr>
        <tr><td>treesitter-context</td><td>0.177603ms</td><td>0.356747ms</td><td>0.534350ms</td></tr>
        <tr><td>trouble.colors</td><td>0.116381ms</td><td>0.152210ms</td><td>0.268591ms</td></tr>
        <tr><td>trouble.config</td><td>0.120217ms</td><td>0.141647ms</td><td>0.261864ms</td></tr>
        <tr><td>trouble.folds</td><td>0.124783ms</td><td>0.059123ms</td><td>0.183906ms</td></tr>
        <tr><td>trouble.providers.lsp</td><td>0.115550ms</td><td>0.315024ms</td><td>0.430574ms</td></tr>
        <tr><td>trouble.providers.qf</td><td>0.117722ms</td><td>0.118342ms</td><td>0.236064ms</td></tr>
        <tr><td>trouble.providers.telescope</td><td>0.128268ms</td><td>0.186273ms</td><td>0.314541ms</td></tr>
        <tr><td>trouble.providers</td><td>0.832562ms</td><td>0.141537ms</td><td>0.974099ms</td></tr>
        <tr><td>trouble.renderer</td><td>0.122382ms</td><td>0.264475ms</td><td>0.386857ms</td></tr>
        <tr><td>trouble.text</td><td>0.119780ms</td><td>0.131700ms</td><td>0.251480ms</td></tr>
        <tr><td>trouble.util</td><td>0.116005ms</td><td>0.337438ms</td><td>0.453443ms</td></tr>
        <tr><td>trouble.view</td><td>0.107589ms</td><td>0.729242ms</td><td>0.836831ms</td></tr>
        <tr><td>trouble</td><td>0.795347ms</td><td>0.290433ms</td><td>1.085780ms</td></tr>
        <tr><td>vim.F</td><td>0.274601ms</td><td>0.029466ms</td><td>0.304067ms</td></tr>
        <tr><td>vim.highlight</td><td>0.301847ms</td><td>0.088167ms</td><td>0.390014ms</td></tr>
        <tr><td>vim.lsp.buf</td><td>0.271953ms</td><td>0.284026ms</td><td>0.555979ms</td></tr>
        <tr><td>vim.lsp.codelens</td><td>0.312490ms</td><td>0.203744ms</td><td>0.516234ms</td></tr>
        <tr><td>vim.lsp.diagnostic</td><td>0.314653ms</td><td>0.747524ms</td><td>1.062177ms</td></tr>
        <tr><td>vim.lsp.handlers</td><td>0.282712ms</td><td>0.332384ms</td><td>0.615096ms</td></tr>
        <tr><td>vim.lsp.log</td><td>0.282614ms</td><td>0.077082ms</td><td>0.359696ms</td></tr>
        <tr><td>vim.lsp.protocol</td><td>0.299905ms</td><td>0.304496ms</td><td>0.604401ms</td></tr>
        <tr><td>vim.lsp.rpc</td><td>0.280200ms</td><td>0.357264ms</td><td>0.637464ms</td></tr>
        <tr><td>vim.lsp.util</td><td>0.297893ms</td><td>1.172185ms</td><td>1.470078ms</td></tr>
        <tr><td>vim.lsp</td><td>0.265146ms</td><td>0.792262ms</td><td>1.057408ms</td></tr>
        <tr><td>vim.treesitter.highlighter</td><td>0.334343ms</td><td>0.211987ms</td><td>0.546330ms</td></tr>
        <tr><td>vim.treesitter.language</td><td>0.311163ms</td><td>0.038987ms</td><td>0.350150ms</td></tr>
        <tr><td>vim.treesitter.languagetree</td><td>0.369525ms</td><td>0.272684ms</td><td>0.642209ms</td></tr>
        <tr><td>vim.treesitter.query</td><td>0.298694ms</td><td>0.320573ms</td><td>0.619267ms</td></tr>
        <tr><td>vim.treesitter</td><td>0.315145ms</td><td>0.090607ms</td><td>0.405752ms</td></tr>
        <tr><td>vim.uri</td><td>0.282207ms</td><td>0.086028ms</td><td>0.368235ms</td></tr>
    </tbody>
</table>
</details>

Total resolve: 68.108365ms, total load: 56.954069ms

<details>
<summary>With reduced runtimepath</summary>

<table>
    <thead>
        <tr><th>Module</th><th>Resolve</th><th>Load</th><th>Total</th></tr>
    </thead>
    <tbody>
        <tr><td>bufferline.buffers</td><td>0.024142ms</td><td>0.087075ms</td><td>0.111217ms</td></tr>
        <tr><td>bufferline.colors</td><td>0.065238ms</td><td>0.588626ms</td><td>0.653864ms</td></tr>
        <tr><td>bufferline.config</td><td>0.026752ms</td><td>0.276576ms</td><td>0.303328ms</td></tr>
        <tr><td>bufferline.constants</td><td>0.031421ms</td><td>0.027580ms</td><td>0.059001ms</td></tr>
        <tr><td>bufferline.custom_area</td><td>0.025813ms</td><td>0.056482ms</td><td>0.082295ms</td></tr>
        <tr><td>bufferline.diagnostics</td><td>0.032609ms</td><td>0.089198ms</td><td>0.121807ms</td></tr>
        <tr><td>bufferline.duplicates</td><td>0.025082ms</td><td>0.071492ms</td><td>0.096574ms</td></tr>
        <tr><td>bufferline.highlights</td><td>0.115814ms</td><td>1.661648ms</td><td>1.777462ms</td></tr>
        <tr><td>bufferline.numbers</td><td>0.025374ms</td><td>0.108112ms</td><td>0.133486ms</td></tr>
        <tr><td>bufferline.offset</td><td>0.027177ms</td><td>0.132531ms</td><td>0.159708ms</td></tr>
        <tr><td>bufferline.pick</td><td>0.030798ms</td><td>0.069584ms</td><td>0.100382ms</td></tr>
        <tr><td>bufferline.sorters</td><td>0.037868ms</td><td>0.104616ms</td><td>0.142484ms</td></tr>
        <tr><td>bufferline.tabs</td><td>0.034044ms</td><td>0.073617ms</td><td>0.107661ms</td></tr>
        <tr><td>bufferline.utils</td><td>0.024159ms</td><td>0.147643ms</td><td>0.171802ms</td></tr>
        <tr><td>bufferline/constants</td><td>0.024380ms</td><td>0.024919ms</td><td>0.049299ms</td></tr>
        <tr><td>bufferline</td><td>0.058271ms</td><td>0.760228ms</td><td>0.818499ms</td></tr>
        <tr><td>cleanfold</td><td>0.037696ms</td><td>0.095122ms</td><td>0.132818ms</td></tr>
        <tr><td>cmp.autocmd</td><td>0.099647ms</td><td>0.040348ms</td><td>0.139995ms</td></tr>
        <tr><td>cmp.config.compare</td><td>0.083547ms</td><td>0.060717ms</td><td>0.144264ms</td></tr>
        <tr><td>cmp.config.default</td><td>0.086109ms</td><td>0.063362ms</td><td>0.149471ms</td></tr>
        <tr><td>cmp.config</td><td>0.080480ms</td><td>0.058701ms</td><td>0.139181ms</td></tr>
        <tr><td>cmp.context</td><td>0.085844ms</td><td>0.121332ms</td><td>0.207176ms</td></tr>
        <tr><td>cmp.core</td><td>0.080110ms</td><td>0.273187ms</td><td>0.353297ms</td></tr>
        <tr><td>cmp.entry</td><td>0.081330ms</td><td>0.288710ms</td><td>0.370040ms</td></tr>
        <tr><td>cmp.float</td><td>0.106399ms</td><td>0.167352ms</td><td>0.273751ms</td></tr>
        <tr><td>cmp.mapping</td><td>0.079923ms</td><td>0.068593ms</td><td>0.148516ms</td></tr>
        <tr><td>cmp.matcher</td><td>0.081619ms</td><td>0.137261ms</td><td>0.218880ms</td></tr>
        <tr><td>cmp.menu</td><td>0.080603ms</td><td>0.172694ms</td><td>0.253297ms</td></tr>
        <tr><td>cmp.source</td><td>0.089229ms</td><td>0.256967ms</td><td>0.346196ms</td></tr>
        <tr><td>cmp.types.cmp</td><td>0.078695ms</td><td>0.032195ms</td><td>0.110890ms</td></tr>
        <tr><td>cmp.types.lsp</td><td>0.097987ms</td><td>0.090137ms</td><td>0.188124ms</td></tr>
        <tr><td>cmp.types.vim</td><td>0.084890ms</td><td>0.014375ms</td><td>0.099265ms</td></tr>
        <tr><td>cmp.types</td><td>0.594839ms</td><td>0.020819ms</td><td>0.615658ms</td></tr>
        <tr><td>cmp.utils.async</td><td>0.092442ms</td><td>0.058766ms</td><td>0.151208ms</td></tr>
        <tr><td>cmp.utils.cache</td><td>0.094490ms</td><td>0.041218ms</td><td>0.135708ms</td></tr>
        <tr><td>cmp.utils.char</td><td>0.095172ms</td><td>0.083701ms</td><td>0.178873ms</td></tr>
        <tr><td>cmp.utils.check</td><td>0.092468ms</td><td>0.031202ms</td><td>0.123670ms</td></tr>
        <tr><td>cmp.utils.debug</td><td>0.101528ms</td><td>0.027757ms</td><td>0.129285ms</td></tr>
        <tr><td>cmp.utils.keymap</td><td>0.090326ms</td><td>0.164131ms</td><td>0.254457ms</td></tr>
        <tr><td>cmp.utils.misc</td><td>0.091474ms</td><td>0.100022ms</td><td>0.191496ms</td></tr>
        <tr><td>cmp.utils.pattern</td><td>0.094489ms</td><td>0.030020ms</td><td>0.124509ms</td></tr>
        <tr><td>cmp.utils.str</td><td>0.088592ms</td><td>0.167706ms</td><td>0.256298ms</td></tr>
        <tr><td>cmp_buffer.buffer</td><td>0.031497ms</td><td>0.114904ms</td><td>0.146401ms</td></tr>
        <tr><td>cmp_buffer</td><td>0.516340ms</td><td>0.096324ms</td><td>0.612664ms</td></tr>
        <tr><td>cmp_luasnip</td><td>0.590500ms</td><td>0.141694ms</td><td>0.732194ms</td></tr>
        <tr><td>cmp_nvim_lsp.source</td><td>0.046699ms</td><td>0.099069ms</td><td>0.145768ms</td></tr>
        <tr><td>cmp_nvim_lsp</td><td>0.528759ms</td><td>0.077387ms</td><td>0.606146ms</td></tr>
        <tr><td>cmp_nvim_lua</td><td>0.522835ms</td><td>0.087704ms</td><td>0.610539ms</td></tr>
        <tr><td>cmp_path</td><td>0.524256ms</td><td>0.160231ms</td><td>0.684487ms</td></tr>
        <tr><td>cmp</td><td>0.542634ms</td><td>0.100938ms</td><td>0.643572ms</td></tr>
        <tr><td>colorizer/nvim</td><td>0.109097ms</td><td>0.189990ms</td><td>0.299087ms</td></tr>
        <tr><td>colorizer/trie</td><td>0.147314ms</td><td>0.168703ms</td><td>0.316017ms</td></tr>
        <tr><td>colorizer</td><td>0.099490ms</td><td>0.509281ms</td><td>0.608771ms</td></tr>
        <tr><td>compe_tmux.source</td><td>0.067131ms</td><td>0.076601ms</td><td>0.143732ms</td></tr>
        <tr><td>compe_tmux.tmux</td><td>0.047828ms</td><td>0.082609ms</td><td>0.130437ms</td></tr>
        <tr><td>compe_tmux.utils</td><td>0.050929ms</td><td>0.034042ms</td><td>0.084971ms</td></tr>
        <tr><td>compe_tmux</td><td>0.541030ms</td><td>0.031311ms</td><td>0.572341ms</td></tr>
        <tr><td>foldsigns</td><td>0.067029ms</td><td>0.096790ms</td><td>0.163819ms</td></tr>
        <tr><td>fzf_lib</td><td>0.146020ms</td><td>0.068963ms</td><td>0.214983ms</td></tr>
        <tr><td>gitsigns.cache</td><td>0.362919ms</td><td>0.525940ms</td><td>0.888859ms</td></tr>
        <tr><td>gitsigns.config</td><td>0.243011ms</td><td>0.754463ms</td><td>0.997474ms</td></tr>
        <tr><td>gitsigns.current_line_blame</td><td>0.437169ms</td><td>0.556336ms</td><td>0.993505ms</td></tr>
        <tr><td>gitsigns.debounce</td><td>0.235829ms</td><td>0.591927ms</td><td>0.827756ms</td></tr>
        <tr><td>gitsigns.debug</td><td>0.328197ms</td><td>0.477773ms</td><td>0.805970ms</td></tr>
        <tr><td>gitsigns.git</td><td>0.319703ms</td><td>0.677741ms</td><td>0.997444ms</td></tr>
        <tr><td>gitsigns.highlight</td><td>0.353525ms</td><td>0.567316ms</td><td>0.920841ms</td></tr>
        <tr><td>gitsigns.hunks</td><td>0.350691ms</td><td>0.566001ms</td><td>0.916692ms</td></tr>
        <tr><td>gitsigns.manager</td><td>0.341269ms</td><td>0.519550ms</td><td>0.860819ms</td></tr>
        <tr><td>gitsigns.signs</td><td>0.426459ms</td><td>0.546394ms</td><td>0.972853ms</td></tr>
        <tr><td>gitsigns.status</td><td>0.501941ms</td><td>0.526907ms</td><td>1.028848ms</td></tr>
        <tr><td>gitsigns.util</td><td>0.210327ms</td><td>0.572105ms</td><td>0.782432ms</td></tr>
        <tr><td>gitsigns</td><td>0.275128ms</td><td>0.715549ms</td><td>0.990677ms</td></tr>
        <tr><td>lspconfig.ui.windows</td><td>0.103825ms</td><td>0.108260ms</td><td>0.212085ms</td></tr>
        <tr><td>lspconfig/configs</td><td>0.109088ms</td><td>0.180962ms</td><td>0.290050ms</td></tr>
        <tr><td>lspconfig/util</td><td>0.090642ms</td><td>0.258532ms</td><td>0.349174ms</td></tr>
        <tr><td>lspconfig</td><td>0.117635ms</td><td>0.086324ms</td><td>0.203959ms</td></tr>
        <tr><td>lspinstall/servers/angular</td><td>0.113813ms</td><td>0.022448ms</td><td>0.136261ms</td></tr>
        <tr><td>lspinstall/servers/bash</td><td>0.151361ms</td><td>0.036614ms</td><td>0.187975ms</td></tr>
        <tr><td>lspinstall/servers/clojure</td><td>0.119435ms</td><td>0.023358ms</td><td>0.142793ms</td></tr>
        <tr><td>lspinstall/servers/cmake</td><td>0.099555ms</td><td>0.018155ms</td><td>0.117710ms</td></tr>
        <tr><td>lspinstall/servers/cpp</td><td>0.108859ms</td><td>0.030185ms</td><td>0.139044ms</td></tr>
        <tr><td>lspinstall/servers/csharp</td><td>0.121643ms</td><td>0.037621ms</td><td>0.159264ms</td></tr>
        <tr><td>lspinstall/servers/css</td><td>0.116422ms</td><td>0.026799ms</td><td>0.143221ms</td></tr>
        <tr><td>lspinstall/servers/deno</td><td>0.097186ms</td><td>0.020918ms</td><td>0.118104ms</td></tr>
        <tr><td>lspinstall/servers/diagnosticls</td><td>0.113099ms</td><td>0.017516ms</td><td>0.130615ms</td></tr>
        <tr><td>lspinstall/servers/dockerfile</td><td>0.106505ms</td><td>0.019209ms</td><td>0.125714ms</td></tr>
        <tr><td>lspinstall/servers/efm</td><td>0.098397ms</td><td>0.016412ms</td><td>0.114809ms</td></tr>
        <tr><td>lspinstall/servers/elixir</td><td>0.107312ms</td><td>0.018782ms</td><td>0.126094ms</td></tr>
        <tr><td>lspinstall/servers/elm</td><td>0.099461ms</td><td>0.023743ms</td><td>0.123204ms</td></tr>
        <tr><td>lspinstall/servers/ember</td><td>0.113416ms</td><td>0.019405ms</td><td>0.132821ms</td></tr>
        <tr><td>lspinstall/servers/fortran</td><td>0.123321ms</td><td>0.020111ms</td><td>0.143432ms</td></tr>
        <tr><td>lspinstall/servers/go</td><td>0.099142ms</td><td>0.018114ms</td><td>0.117256ms</td></tr>
        <tr><td>lspinstall/servers/graphql</td><td>0.131506ms</td><td>0.028137ms</td><td>0.159643ms</td></tr>
        <tr><td>lspinstall/servers/haskell</td><td>0.103350ms</td><td>0.018438ms</td><td>0.121788ms</td></tr>
        <tr><td>lspinstall/servers/html</td><td>0.098430ms</td><td>0.029734ms</td><td>0.128164ms</td></tr>
        <tr><td>lspinstall/servers/java</td><td>0.097525ms</td><td>0.079231ms</td><td>0.176756ms</td></tr>
        <tr><td>lspinstall/servers/json</td><td>0.102573ms</td><td>0.025866ms</td><td>0.128439ms</td></tr>
        <tr><td>lspinstall/servers/kotlin</td><td>0.099422ms</td><td>0.018714ms</td><td>0.118136ms</td></tr>
        <tr><td>lspinstall/servers/latex</td><td>0.099763ms</td><td>0.018652ms</td><td>0.118415ms</td></tr>
        <tr><td>lspinstall/servers/lua</td><td>0.101885ms</td><td>0.022724ms</td><td>0.124609ms</td></tr>
        <tr><td>lspinstall/servers/php</td><td>0.097808ms</td><td>0.018476ms</td><td>0.116284ms</td></tr>
        <tr><td>lspinstall/servers/puppet</td><td>0.100262ms</td><td>0.017709ms</td><td>0.117971ms</td></tr>
        <tr><td>lspinstall/servers/purescript</td><td>0.103902ms</td><td>0.028484ms</td><td>0.132386ms</td></tr>
        <tr><td>lspinstall/servers/python</td><td>0.109806ms</td><td>0.019118ms</td><td>0.128924ms</td></tr>
        <tr><td>lspinstall/servers/rome</td><td>0.110277ms</td><td>0.030327ms</td><td>0.140604ms</td></tr>
        <tr><td>lspinstall/servers/ruby</td><td>0.099454ms</td><td>0.019118ms</td><td>0.118572ms</td></tr>
        <tr><td>lspinstall/servers/rust</td><td>0.098665ms</td><td>0.020193ms</td><td>0.118858ms</td></tr>
        <tr><td>lspinstall/servers/svelte</td><td>0.100098ms</td><td>0.027600ms</td><td>0.127698ms</td></tr>
        <tr><td>lspinstall/servers/tailwindcss</td><td>0.104957ms</td><td>0.016134ms</td><td>0.121091ms</td></tr>
        <tr><td>lspinstall/servers/terraform</td><td>0.110115ms</td><td>0.025983ms</td><td>0.136098ms</td></tr>
        <tr><td>lspinstall/servers/typescript</td><td>0.112367ms</td><td>0.016961ms</td><td>0.129328ms</td></tr>
        <tr><td>lspinstall/servers/vim</td><td>0.099954ms</td><td>0.017125ms</td><td>0.117079ms</td></tr>
        <tr><td>lspinstall/servers/vue</td><td>0.108317ms</td><td>0.018897ms</td><td>0.127214ms</td></tr>
        <tr><td>lspinstall/servers/yaml</td><td>0.099349ms</td><td>0.019443ms</td><td>0.118792ms</td></tr>
        <tr><td>lspinstall/servers</td><td>0.122064ms</td><td>0.066008ms</td><td>0.188072ms</td></tr>
        <tr><td>lspinstall/util</td><td>0.098740ms</td><td>0.042679ms</td><td>0.141419ms</td></tr>
        <tr><td>lspinstall</td><td>0.115485ms</td><td>0.117957ms</td><td>0.233442ms</td></tr>
        <tr><td>lspkind</td><td>0.618609ms</td><td>0.083375ms</td><td>0.701984ms</td></tr>
        <tr><td>luasnip.config</td><td>0.019899ms</td><td>0.242240ms</td><td>0.262139ms</td></tr>
        <tr><td>luasnip.nodes.choiceNode</td><td>0.021969ms</td><td>0.158023ms</td><td>0.179992ms</td></tr>
        <tr><td>luasnip.nodes.dynamicNode</td><td>0.031339ms</td><td>0.127597ms</td><td>0.158936ms</td></tr>
        <tr><td>luasnip.nodes.functionNode</td><td>0.029619ms</td><td>0.086096ms</td><td>0.115715ms</td></tr>
        <tr><td>luasnip.nodes.insertNode</td><td>0.020500ms</td><td>0.128444ms</td><td>0.148944ms</td></tr>
        <tr><td>luasnip.nodes.node</td><td>0.024696ms</td><td>0.109964ms</td><td>0.134660ms</td></tr>
        <tr><td>luasnip.nodes.snippet</td><td>0.024234ms</td><td>0.522767ms</td><td>0.547001ms</td></tr>
        <tr><td>luasnip.nodes.textNode</td><td>0.024807ms</td><td>0.041911ms</td><td>0.066718ms</td></tr>
        <tr><td>luasnip.session</td><td>0.020654ms</td><td>0.010978ms</td><td>0.031632ms</td></tr>
        <tr><td>luasnip.util.environ</td><td>0.021449ms</td><td>0.131778ms</td><td>0.153227ms</td></tr>
        <tr><td>luasnip.util.events</td><td>0.023979ms</td><td>0.025281ms</td><td>0.049260ms</td></tr>
        <tr><td>luasnip.util.functions</td><td>0.023537ms</td><td>0.022666ms</td><td>0.046203ms</td></tr>
        <tr><td>luasnip.util.mark</td><td>0.019835ms</td><td>0.104460ms</td><td>0.124295ms</td></tr>
        <tr><td>luasnip.util.parser</td><td>0.021500ms</td><td>0.276620ms</td><td>0.298120ms</td></tr>
        <tr><td>luasnip.util.pattern_tokenizer</td><td>0.027231ms</td><td>0.088304ms</td><td>0.115535ms</td></tr>
        <tr><td>luasnip.util.types</td><td>0.020051ms</td><td>0.021142ms</td><td>0.041193ms</td></tr>
        <tr><td>luasnip.util.util</td><td>0.018998ms</td><td>0.337294ms</td><td>0.356292ms</td></tr>
        <tr><td>luasnip</td><td>0.539174ms</td><td>0.299332ms</td><td>0.838506ms</td></tr>
        <tr><td>null-ls.builtins.code-actions</td><td>0.101652ms</td><td>0.063032ms</td><td>0.164684ms</td></tr>
        <tr><td>null-ls.builtins.diagnostics</td><td>0.087050ms</td><td>0.407843ms</td><td>0.494893ms</td></tr>
        <tr><td>null-ls.builtins.formatting</td><td>0.102113ms</td><td>0.471555ms</td><td>0.573668ms</td></tr>
        <tr><td>null-ls.builtins.test</td><td>0.083093ms</td><td>0.143603ms</td><td>0.226696ms</td></tr>
        <tr><td>null-ls.builtins</td><td>0.602941ms</td><td>0.076765ms</td><td>0.679706ms</td></tr>
        <tr><td>null-ls.code-actions</td><td>0.082187ms</td><td>0.065517ms</td><td>0.147704ms</td></tr>
        <tr><td>null-ls.config</td><td>0.078337ms</td><td>0.329027ms</td><td>0.407364ms</td></tr>
        <tr><td>null-ls.diagnostics</td><td>0.099645ms</td><td>0.109067ms</td><td>0.208712ms</td></tr>
        <tr><td>null-ls.formatting</td><td>0.081440ms</td><td>2.501997ms</td><td>2.583437ms</td></tr>
        <tr><td>null-ls.generators</td><td>0.078833ms</td><td>0.203372ms</td><td>0.282205ms</td></tr>
        <tr><td>null-ls.handlers</td><td>0.080301ms</td><td>0.067006ms</td><td>0.147307ms</td></tr>
        <tr><td>null-ls.helpers</td><td>0.078331ms</td><td>0.433748ms</td><td>0.512079ms</td></tr>
        <tr><td>null-ls.info</td><td>0.096216ms</td><td>0.124613ms</td><td>0.220829ms</td></tr>
        <tr><td>null-ls.logger</td><td>0.085100ms</td><td>0.036844ms</td><td>0.121944ms</td></tr>
        <tr><td>null-ls.loop</td><td>0.083176ms</td><td>0.156637ms</td><td>0.239813ms</td></tr>
        <tr><td>null-ls.lspconfig</td><td>0.104403ms</td><td>0.108860ms</td><td>0.213263ms</td></tr>
        <tr><td>null-ls.methods</td><td>0.097723ms</td><td>0.045125ms</td><td>0.142848ms</td></tr>
        <tr><td>null-ls.rpc</td><td>0.081435ms</td><td>0.136117ms</td><td>0.217552ms</td></tr>
        <tr><td>null-ls.state</td><td>0.079789ms</td><td>0.090256ms</td><td>0.170045ms</td></tr>
        <tr><td>null-ls.utils</td><td>0.095074ms</td><td>0.176144ms</td><td>0.271218ms</td></tr>
        <tr><td>null-ls</td><td>0.643104ms</td><td>0.060956ms</td><td>0.704060ms</td></tr>
        <tr><td>nvim-treesitter-playground</td><td>0.145890ms</td><td>0.146784ms</td><td>0.292674ms</td></tr>
        <tr><td>nvim-treesitter.caching</td><td>0.119343ms</td><td>0.060540ms</td><td>0.179883ms</td></tr>
        <tr><td>nvim-treesitter.configs</td><td>0.109959ms</td><td>0.384223ms</td><td>0.494182ms</td></tr>
        <tr><td>nvim-treesitter.info</td><td>0.118572ms</td><td>0.159906ms</td><td>0.278478ms</td></tr>
        <tr><td>nvim-treesitter.install</td><td>0.138380ms</td><td>0.482649ms</td><td>0.621029ms</td></tr>
        <tr><td>nvim-treesitter.parsers</td><td>0.150296ms</td><td>0.550505ms</td><td>0.700801ms</td></tr>
        <tr><td>nvim-treesitter.query_predicates</td><td>0.154577ms</td><td>0.182715ms</td><td>0.337292ms</td></tr>
        <tr><td>nvim-treesitter.query</td><td>0.197056ms</td><td>0.291868ms</td><td>0.488924ms</td></tr>
        <tr><td>nvim-treesitter.shell_command_selectors</td><td>0.150823ms</td><td>0.389069ms</td><td>0.539892ms</td></tr>
        <tr><td>nvim-treesitter.ts_utils</td><td>0.122162ms</td><td>0.313360ms</td><td>0.435522ms</td></tr>
        <tr><td>nvim-treesitter.tsrange</td><td>0.116247ms</td><td>0.158595ms</td><td>0.274842ms</td></tr>
        <tr><td>nvim-treesitter.utils</td><td>0.126973ms</td><td>0.203408ms</td><td>0.330381ms</td></tr>
        <tr><td>nvim-treesitter</td><td>0.123309ms</td><td>0.109718ms</td><td>0.233027ms</td></tr>
        <tr><td>nvim-web-devicons</td><td>0.134993ms</td><td>0.545327ms</td><td>0.680320ms</td></tr>
        <tr><td>octo.base64</td><td>0.022251ms</td><td>0.178191ms</td><td>0.200442ms</td></tr>
        <tr><td>octo.colors</td><td>0.027501ms</td><td>0.242183ms</td><td>0.269684ms</td></tr>
        <tr><td>octo.completion</td><td>0.018860ms</td><td>0.085188ms</td><td>0.104048ms</td></tr>
        <tr><td>octo.config</td><td>0.016425ms</td><td>0.105354ms</td><td>0.121779ms</td></tr>
        <tr><td>octo.constants</td><td>0.022025ms</td><td>0.040543ms</td><td>0.062568ms</td></tr>
        <tr><td>octo.date</td><td>0.015860ms</td><td>0.964797ms</td><td>0.980657ms</td></tr>
        <tr><td>octo.folds</td><td>0.016690ms</td><td>0.028286ms</td><td>0.044976ms</td></tr>
        <tr><td>octo.gh</td><td>0.017949ms</td><td>0.086958ms</td><td>0.104907ms</td></tr>
        <tr><td>octo.graphql</td><td>0.019598ms</td><td>0.297414ms</td><td>0.317012ms</td></tr>
        <tr><td>octo.mappings</td><td>0.018708ms</td><td>0.154979ms</td><td>0.173687ms</td></tr>
        <tr><td>octo.model.body-metadata</td><td>0.029718ms</td><td>0.032678ms</td><td>0.062396ms</td></tr>
        <tr><td>octo.model.octo-buffer</td><td>0.019460ms</td><td>0.537845ms</td><td>0.557305ms</td></tr>
        <tr><td>octo.model.thread-metadata</td><td>0.021694ms</td><td>0.024518ms</td><td>0.046212ms</td></tr>
        <tr><td>octo.model.title-metadata</td><td>0.016967ms</td><td>0.022489ms</td><td>0.039456ms</td></tr>
        <tr><td>octo.reviews.file-entry</td><td>0.021431ms</td><td>0.288190ms</td><td>0.309621ms</td></tr>
        <tr><td>octo.reviews.file-panel</td><td>0.017990ms</td><td>0.353803ms</td><td>0.371793ms</td></tr>
        <tr><td>octo.reviews.layout</td><td>0.018132ms</td><td>0.194960ms</td><td>0.213092ms</td></tr>
        <tr><td>octo.reviews.renderer</td><td>0.018912ms</td><td>0.083523ms</td><td>0.102435ms</td></tr>
        <tr><td>octo.reviews</td><td>0.016957ms</td><td>0.444379ms</td><td>0.461336ms</td></tr>
        <tr><td>octo.signs</td><td>0.017471ms</td><td>0.075055ms</td><td>0.092526ms</td></tr>
        <tr><td>octo.ui.bubbles</td><td>0.015735ms</td><td>0.074110ms</td><td>0.089845ms</td></tr>
        <tr><td>octo.utils</td><td>0.016355ms</td><td>0.754394ms</td><td>0.770749ms</td></tr>
        <tr><td>octo.window</td><td>0.016899ms</td><td>0.190229ms</td><td>0.207128ms</td></tr>
        <tr><td>octo.writers</td><td>0.017493ms</td><td>1.008661ms</td><td>1.026154ms</td></tr>
        <tr><td>octo</td><td>0.016104ms</td><td>0.247166ms</td><td>0.263270ms</td></tr>
        <tr><td>packer.load</td><td>0.177614ms</td><td>0.163326ms</td><td>0.340940ms</td></tr>
        <tr><td>plenary.async.async</td><td>0.179188ms</td><td>0.082880ms</td><td>0.262068ms</td></tr>
        <tr><td>plenary.async.control</td><td>0.130136ms</td><td>0.136668ms</td><td>0.266804ms</td></tr>
        <tr><td>plenary.async.structs</td><td>0.129710ms</td><td>0.065059ms</td><td>0.194769ms</td></tr>
        <tr><td>plenary.async.util</td><td>0.121026ms</td><td>0.097012ms</td><td>0.218038ms</td></tr>
        <tr><td>plenary.async_lib.api</td><td>0.125435ms</td><td>0.029839ms</td><td>0.155274ms</td></tr>
        <tr><td>plenary.async_lib.async</td><td>0.139630ms</td><td>0.202657ms</td><td>0.342287ms</td></tr>
        <tr><td>plenary.async_lib.lsp</td><td>0.132682ms</td><td>0.026156ms</td><td>0.158838ms</td></tr>
        <tr><td>plenary.async_lib.structs</td><td>0.144202ms</td><td>0.084566ms</td><td>0.228768ms</td></tr>
        <tr><td>plenary.async_lib.tests</td><td>0.128185ms</td><td>0.022409ms</td><td>0.150594ms</td></tr>
        <tr><td>plenary.async_lib.util</td><td>0.139156ms</td><td>0.291280ms</td><td>0.430436ms</td></tr>
        <tr><td>plenary.async_lib.uv_async</td><td>0.132875ms</td><td>0.178208ms</td><td>0.311083ms</td></tr>
        <tr><td>plenary.async_lib</td><td>0.622319ms</td><td>0.067232ms</td><td>0.689551ms</td></tr>
        <tr><td>plenary.bit</td><td>0.141775ms</td><td>0.226418ms</td><td>0.368193ms</td></tr>
        <tr><td>plenary.errors</td><td>0.127876ms</td><td>0.025394ms</td><td>0.153270ms</td></tr>
        <tr><td>plenary.functional</td><td>0.130023ms</td><td>0.059557ms</td><td>0.189580ms</td></tr>
        <tr><td>plenary.job</td><td>0.133660ms</td><td>0.403723ms</td><td>0.537383ms</td></tr>
        <tr><td>plenary.log</td><td>0.117279ms</td><td>0.166041ms</td><td>0.283320ms</td></tr>
        <tr><td>plenary.path</td><td>0.119257ms</td><td>0.507160ms</td><td>0.626417ms</td></tr>
        <tr><td>plenary.strings</td><td>0.140266ms</td><td>0.131006ms</td><td>0.271272ms</td></tr>
        <tr><td>plenary.tbl</td><td>0.112866ms</td><td>0.039096ms</td><td>0.151962ms</td></tr>
        <tr><td>plenary.vararg.rotate</td><td>0.134745ms</td><td>0.077372ms</td><td>0.212117ms</td></tr>
        <tr><td>plenary.vararg</td><td>0.645962ms</td><td>0.018593ms</td><td>0.664555ms</td></tr>
        <tr><td>spaceless</td><td>0.163511ms</td><td>0.082662ms</td><td>0.246173ms</td></tr>
        <tr><td>spellsitter</td><td>0.361876ms</td><td>0.657404ms</td><td>1.019280ms</td></tr>
        <tr><td>symbols-outline.config</td><td>0.142297ms</td><td>0.105062ms</td><td>0.247359ms</td></tr>
        <tr><td>symbols-outline.markdown</td><td>0.143558ms</td><td>0.040536ms</td><td>0.184094ms</td></tr>
        <tr><td>symbols-outline.parser</td><td>0.171564ms</td><td>0.338453ms</td><td>0.510017ms</td></tr>
        <tr><td>symbols-outline.symbols</td><td>0.176726ms</td><td>0.036425ms</td><td>0.213151ms</td></tr>
        <tr><td>symbols-outline.ui</td><td>0.153168ms</td><td>0.072949ms</td><td>0.226117ms</td></tr>
        <tr><td>symbols-outline.utils.init</td><td>0.149088ms</td><td>0.051912ms</td><td>0.201000ms</td></tr>
        <tr><td>symbols-outline.utils.lsp_utils</td><td>0.184382ms</td><td>0.033743ms</td><td>0.218125ms</td></tr>
        <tr><td>symbols-outline.view</td><td>0.158047ms</td><td>0.058443ms</td><td>0.216490ms</td></tr>
        <tr><td>symbols-outline.writer</td><td>0.146275ms</td><td>0.056341ms</td><td>0.202616ms</td></tr>
        <tr><td>symbols-outline</td><td>0.157327ms</td><td>0.588785ms</td><td>0.746112ms</td></tr>
        <tr><td>telescope._extensions.fzf</td><td>0.188353ms</td><td>0.141791ms</td><td>0.330144ms</td></tr>
        <tr><td>telescope._extensions</td><td>0.662232ms</td><td>0.046337ms</td><td>0.708569ms</td></tr>
        <tr><td>telescope.config</td><td>0.146316ms</td><td>0.235658ms</td><td>0.381974ms</td></tr>
        <tr><td>telescope.deprecated</td><td>0.181079ms</td><td>0.057196ms</td><td>0.238275ms</td></tr>
        <tr><td>telescope.log</td><td>0.153879ms</td><td>0.023012ms</td><td>0.176891ms</td></tr>
        <tr><td>telescope.sorters</td><td>0.159688ms</td><td>0.397819ms</td><td>0.557507ms</td></tr>
        <tr><td>telescope.utils</td><td>0.162827ms</td><td>0.411137ms</td><td>0.573964ms</td></tr>
        <tr><td>telescope</td><td>0.960782ms</td><td>0.080267ms</td><td>1.041049ms</td></tr>
        <tr><td>treesitter-context.utils</td><td>0.162420ms</td><td>0.133792ms</td><td>0.296212ms</td></tr>
        <tr><td>treesitter-context</td><td>0.130868ms</td><td>0.929447ms</td><td>1.060315ms</td></tr>
        <tr><td>trouble.colors</td><td>0.160137ms</td><td>0.067051ms</td><td>0.227188ms</td></tr>
        <tr><td>trouble.config</td><td>0.166175ms</td><td>0.091435ms</td><td>0.257610ms</td></tr>
        <tr><td>trouble.folds</td><td>0.163031ms</td><td>0.041608ms</td><td>0.204639ms</td></tr>
        <tr><td>trouble.providers.lsp</td><td>0.171746ms</td><td>0.131343ms</td><td>0.303089ms</td></tr>
        <tr><td>trouble.providers.qf</td><td>0.164773ms</td><td>0.087112ms</td><td>0.251885ms</td></tr>
        <tr><td>trouble.providers.telescope</td><td>0.176871ms</td><td>0.085431ms</td><td>0.262302ms</td></tr>
        <tr><td>trouble.providers</td><td>0.683404ms</td><td>0.087062ms</td><td>0.770466ms</td></tr>
        <tr><td>trouble.renderer</td><td>0.172769ms</td><td>0.157509ms</td><td>0.330278ms</td></tr>
        <tr><td>trouble.text</td><td>0.174238ms</td><td>0.068453ms</td><td>0.242691ms</td></tr>
        <tr><td>trouble.util</td><td>0.151959ms</td><td>0.231243ms</td><td>0.383202ms</td></tr>
        <tr><td>trouble.view</td><td>0.164633ms</td><td>0.484488ms</td><td>0.649121ms</td></tr>
        <tr><td>trouble</td><td>0.674607ms</td><td>0.233342ms</td><td>0.907949ms</td></tr>
        <tr><td>vim.F</td><td>0.020387ms</td><td>0.029165ms</td><td>0.049552ms</td></tr>
        <tr><td>vim.highlight</td><td>0.019936ms</td><td>0.091714ms</td><td>0.111650ms</td></tr>
        <tr><td>vim.lsp.buf</td><td>0.015468ms</td><td>0.299722ms</td><td>0.315190ms</td></tr>
        <tr><td>vim.lsp.codelens</td><td>0.020409ms</td><td>0.185727ms</td><td>0.206136ms</td></tr>
        <tr><td>vim.lsp.diagnostic</td><td>0.019575ms</td><td>0.792515ms</td><td>0.812090ms</td></tr>
        <tr><td>vim.lsp.handlers</td><td>0.016377ms</td><td>0.339889ms</td><td>0.356266ms</td></tr>
        <tr><td>vim.lsp.log</td><td>0.017201ms</td><td>0.092081ms</td><td>0.109282ms</td></tr>
        <tr><td>vim.lsp.protocol</td><td>0.021492ms</td><td>0.321789ms</td><td>0.343281ms</td></tr>
        <tr><td>vim.lsp.rpc</td><td>0.016629ms</td><td>0.391893ms</td><td>0.408522ms</td></tr>
        <tr><td>vim.lsp.util</td><td>0.022533ms</td><td>1.229670ms</td><td>1.252203ms</td></tr>
        <tr><td>vim.lsp</td><td>0.014237ms</td><td>0.834510ms</td><td>0.848747ms</td></tr>
        <tr><td>vim.treesitter.highlighter</td><td>0.025674ms</td><td>0.597703ms</td><td>0.623377ms</td></tr>
        <tr><td>vim.treesitter.language</td><td>0.019693ms</td><td>0.041021ms</td><td>0.060714ms</td></tr>
        <tr><td>vim.treesitter.languagetree</td><td>0.029201ms</td><td>0.301450ms</td><td>0.330651ms</td></tr>
        <tr><td>vim.treesitter.query</td><td>0.020801ms</td><td>0.328748ms</td><td>0.349549ms</td></tr>
        <tr><td>vim.treesitter</td><td>0.020061ms</td><td>0.093980ms</td><td>0.114041ms</td></tr>
        <tr><td>vim.uri</td><td>0.018153ms</td><td>0.083123ms</td><td>0.101276ms</td></tr>
    </tbody>
</table>
</details>

Total resolve: 39.302892ms, total load: 58.602757ms

<details>
<summary>With cache</summary>

<table>
    <thead>
        <tr><th>Module</th><th>Resolve</th><th>Load</th><th>Total</th></tr>
    </thead>
    <tbody>
        <tr><td>bufferline.buffers</td><td>0.004250ms</td><td>0.012261ms</td><td>0.016511ms</td></tr>
        <tr><td>bufferline.colors</td><td>0.018941ms</td><td>0.008184ms</td><td>0.027125ms</td></tr>
        <tr><td>bufferline.config</td><td>0.005740ms</td><td>0.033685ms</td><td>0.039425ms</td></tr>
        <tr><td>bufferline.constants</td><td>0.008306ms</td><td>0.004448ms</td><td>0.012754ms</td></tr>
        <tr><td>bufferline.custom_area</td><td>0.004360ms</td><td>0.005803ms</td><td>0.010163ms</td></tr>
        <tr><td>bufferline.diagnostics</td><td>0.004011ms</td><td>0.018458ms</td><td>0.022469ms</td></tr>
        <tr><td>bufferline.duplicates</td><td>0.004455ms</td><td>0.006415ms</td><td>0.010870ms</td></tr>
        <tr><td>bufferline.highlights</td><td>0.008616ms</td><td>0.007238ms</td><td>0.015854ms</td></tr>
        <tr><td>bufferline.numbers</td><td>0.005316ms</td><td>0.014697ms</td><td>0.020013ms</td></tr>
        <tr><td>bufferline.offset</td><td>0.007430ms</td><td>0.011513ms</td><td>0.018943ms</td></tr>
        <tr><td>bufferline.pick</td><td>0.006019ms</td><td>0.008048ms</td><td>0.014067ms</td></tr>
        <tr><td>bufferline.sorters</td><td>0.013036ms</td><td>0.011732ms</td><td>0.024768ms</td></tr>
        <tr><td>bufferline.tabs</td><td>0.014849ms</td><td>0.010627ms</td><td>0.025476ms</td></tr>
        <tr><td>bufferline.utils</td><td>0.008699ms</td><td>0.014861ms</td><td>0.023560ms</td></tr>
        <tr><td>bufferline/constants</td><td>0.004990ms</td><td>0.004347ms</td><td>0.009337ms</td></tr>
        <tr><td>bufferline</td><td>0.009618ms</td><td>0.122752ms</td><td>0.132370ms</td></tr>
        <tr><td>cleanfold</td><td>0.007238ms</td><td>0.008114ms</td><td>0.015352ms</td></tr>
        <tr><td>cmp.autocmd</td><td>0.004328ms</td><td>0.004324ms</td><td>0.008652ms</td></tr>
        <tr><td>cmp.config.compare</td><td>0.005582ms</td><td>0.002711ms</td><td>0.008293ms</td></tr>
        <tr><td>cmp.config.default</td><td>0.004320ms</td><td>0.007652ms</td><td>0.011972ms</td></tr>
        <tr><td>cmp.config</td><td>0.006575ms</td><td>0.005180ms</td><td>0.011755ms</td></tr>
        <tr><td>cmp.context</td><td>0.005888ms</td><td>0.008277ms</td><td>0.014165ms</td></tr>
        <tr><td>cmp.core</td><td>0.006343ms</td><td>0.021141ms</td><td>0.027484ms</td></tr>
        <tr><td>cmp.entry</td><td>0.003902ms</td><td>0.020022ms</td><td>0.023924ms</td></tr>
        <tr><td>cmp.float</td><td>0.003957ms</td><td>0.010955ms</td><td>0.014912ms</td></tr>
        <tr><td>cmp.mapping</td><td>0.006338ms</td><td>0.006175ms</td><td>0.012513ms</td></tr>
        <tr><td>cmp.matcher</td><td>0.005356ms</td><td>0.005663ms</td><td>0.011019ms</td></tr>
        <tr><td>cmp.menu</td><td>0.005389ms</td><td>0.021984ms</td><td>0.027373ms</td></tr>
        <tr><td>cmp.source</td><td>0.005651ms</td><td>0.019877ms</td><td>0.025528ms</td></tr>
        <tr><td>cmp.types.cmp</td><td>0.004004ms</td><td>0.003197ms</td><td>0.007201ms</td></tr>
        <tr><td>cmp.types.lsp</td><td>0.003553ms</td><td>0.006521ms</td><td>0.010074ms</td></tr>
        <tr><td>cmp.types.vim</td><td>0.006374ms</td><td>0.000795ms</td><td>0.007169ms</td></tr>
        <tr><td>cmp.types</td><td>0.006090ms</td><td>0.001100ms</td><td>0.007190ms</td></tr>
        <tr><td>cmp.utils.async</td><td>0.003951ms</td><td>0.002941ms</td><td>0.006892ms</td></tr>
        <tr><td>cmp.utils.cache</td><td>0.006372ms</td><td>0.002590ms</td><td>0.008962ms</td></tr>
        <tr><td>cmp.utils.char</td><td>0.004019ms</td><td>0.004410ms</td><td>0.008429ms</td></tr>
        <tr><td>cmp.utils.check</td><td>0.004065ms</td><td>0.003358ms</td><td>0.007423ms</td></tr>
        <tr><td>cmp.utils.debug</td><td>0.005002ms</td><td>0.003482ms</td><td>0.008484ms</td></tr>
        <tr><td>cmp.utils.keymap</td><td>0.003610ms</td><td>0.016605ms</td><td>0.020215ms</td></tr>
        <tr><td>cmp.utils.misc</td><td>0.005452ms</td><td>0.016780ms</td><td>0.022232ms</td></tr>
        <tr><td>cmp.utils.pattern</td><td>0.003632ms</td><td>0.001380ms</td><td>0.005012ms</td></tr>
        <tr><td>cmp.utils.str</td><td>0.003975ms</td><td>0.004904ms</td><td>0.008879ms</td></tr>
        <tr><td>cmp_buffer.buffer</td><td>0.004470ms</td><td>0.009905ms</td><td>0.014375ms</td></tr>
        <tr><td>cmp_buffer</td><td>0.010024ms</td><td>0.020445ms</td><td>0.030469ms</td></tr>
        <tr><td>cmp_luasnip</td><td>0.015840ms</td><td>0.017877ms</td><td>0.033717ms</td></tr>
        <tr><td>cmp_nvim_lsp.source</td><td>0.006289ms</td><td>0.010595ms</td><td>0.016884ms</td></tr>
        <tr><td>cmp_nvim_lsp</td><td>0.007247ms</td><td>0.010260ms</td><td>0.017507ms</td></tr>
        <tr><td>cmp_nvim_lua</td><td>0.008360ms</td><td>0.013666ms</td><td>0.022026ms</td></tr>
        <tr><td>cmp_path</td><td>0.010924ms</td><td>0.014313ms</td><td>0.025237ms</td></tr>
        <tr><td>cmp</td><td>0.006787ms</td><td>0.146396ms</td><td>0.153183ms</td></tr>
        <tr><td>colorizer/nvim</td><td>0.004115ms</td><td>0.019435ms</td><td>0.023550ms</td></tr>
        <tr><td>colorizer/trie</td><td>0.003927ms</td><td>0.012114ms</td><td>0.016041ms</td></tr>
        <tr><td>colorizer</td><td>0.006310ms</td><td>0.032096ms</td><td>0.038406ms</td></tr>
        <tr><td>compe_tmux.source</td><td>0.005527ms</td><td>0.013317ms</td><td>0.018844ms</td></tr>
        <tr><td>compe_tmux.tmux</td><td>0.004339ms</td><td>0.008803ms</td><td>0.013142ms</td></tr>
        <tr><td>compe_tmux.utils</td><td>0.004465ms</td><td>0.005793ms</td><td>0.010258ms</td></tr>
        <tr><td>compe_tmux</td><td>0.016028ms</td><td>0.003595ms</td><td>0.019623ms</td></tr>
        <tr><td>foldsigns</td><td>0.008489ms</td><td>0.006335ms</td><td>0.014824ms</td></tr>
        <tr><td>fzf_lib</td><td>0.006405ms</td><td>0.006197ms</td><td>0.012602ms</td></tr>
        <tr><td>gitsigns.cache</td><td>0.006423ms</td><td>0.004014ms</td><td>0.010437ms</td></tr>
        <tr><td>gitsigns.config</td><td>0.005867ms</td><td>0.031239ms</td><td>0.037106ms</td></tr>
        <tr><td>gitsigns.current_line_blame</td><td>0.023475ms</td><td>0.013864ms</td><td>0.037339ms</td></tr>
        <tr><td>gitsigns.debounce</td><td>0.006431ms</td><td>0.002568ms</td><td>0.008999ms</td></tr>
        <tr><td>gitsigns.debug</td><td>0.009322ms</td><td>0.013121ms</td><td>0.022443ms</td></tr>
        <tr><td>gitsigns.git</td><td>0.006912ms</td><td>0.024315ms</td><td>0.031227ms</td></tr>
        <tr><td>gitsigns.highlight</td><td>0.006821ms</td><td>0.006990ms</td><td>0.013811ms</td></tr>
        <tr><td>gitsigns.hunks</td><td>0.007325ms</td><td>0.009777ms</td><td>0.017102ms</td></tr>
        <tr><td>gitsigns.manager</td><td>0.006440ms</td><td>0.010964ms</td><td>0.017404ms</td></tr>
        <tr><td>gitsigns.signs</td><td>0.007020ms</td><td>0.006197ms</td><td>0.013217ms</td></tr>
        <tr><td>gitsigns.status</td><td>0.008294ms</td><td>0.003510ms</td><td>0.011804ms</td></tr>
        <tr><td>gitsigns.util</td><td>0.009524ms</td><td>0.008127ms</td><td>0.017651ms</td></tr>
        <tr><td>gitsigns</td><td>0.011905ms</td><td>0.027477ms</td><td>0.039382ms</td></tr>
        <tr><td>lspconfig.ui.windows</td><td>0.007511ms</td><td>0.007041ms</td><td>0.014552ms</td></tr>
        <tr><td>lspconfig/configs</td><td>0.006690ms</td><td>0.019841ms</td><td>0.026531ms</td></tr>
        <tr><td>lspconfig/util</td><td>0.004152ms</td><td>0.028304ms</td><td>0.032456ms</td></tr>
        <tr><td>lspconfig</td><td>0.005751ms</td><td>0.010873ms</td><td>0.016624ms</td></tr>
        <tr><td>lspinstall/servers/angular</td><td>0.006350ms</td><td>0.009991ms</td><td>0.016341ms</td></tr>
        <tr><td>lspinstall/servers/bash</td><td>0.009515ms</td><td>0.002998ms</td><td>0.012513ms</td></tr>
        <tr><td>lspinstall/servers/clojure</td><td>0.007717ms</td><td>0.003554ms</td><td>0.011271ms</td></tr>
        <tr><td>lspinstall/servers/cmake</td><td>0.007319ms</td><td>0.002507ms</td><td>0.009826ms</td></tr>
        <tr><td>lspinstall/servers/cpp</td><td>0.008612ms</td><td>0.003202ms</td><td>0.011814ms</td></tr>
        <tr><td>lspinstall/servers/csharp</td><td>0.011234ms</td><td>0.011745ms</td><td>0.022979ms</td></tr>
        <tr><td>lspinstall/servers/css</td><td>0.009829ms</td><td>0.004519ms</td><td>0.014348ms</td></tr>
        <tr><td>lspinstall/servers/deno</td><td>0.005647ms</td><td>0.005145ms</td><td>0.010792ms</td></tr>
        <tr><td>lspinstall/servers/diagnosticls</td><td>0.009121ms</td><td>0.003517ms</td><td>0.012638ms</td></tr>
        <tr><td>lspinstall/servers/dockerfile</td><td>0.009337ms</td><td>0.004054ms</td><td>0.013391ms</td></tr>
        <tr><td>lspinstall/servers/efm</td><td>0.010858ms</td><td>0.003638ms</td><td>0.014496ms</td></tr>
        <tr><td>lspinstall/servers/elixir</td><td>0.011432ms</td><td>0.004966ms</td><td>0.016398ms</td></tr>
        <tr><td>lspinstall/servers/elm</td><td>0.010712ms</td><td>0.007388ms</td><td>0.018100ms</td></tr>
        <tr><td>lspinstall/servers/ember</td><td>0.009217ms</td><td>0.003952ms</td><td>0.013169ms</td></tr>
        <tr><td>lspinstall/servers/fortran</td><td>0.007130ms</td><td>0.003405ms</td><td>0.010535ms</td></tr>
        <tr><td>lspinstall/servers/go</td><td>0.008187ms</td><td>0.005612ms</td><td>0.013799ms</td></tr>
        <tr><td>lspinstall/servers/graphql</td><td>0.012260ms</td><td>0.004077ms</td><td>0.016337ms</td></tr>
        <tr><td>lspinstall/servers/haskell</td><td>0.023437ms</td><td>0.004913ms</td><td>0.028350ms</td></tr>
        <tr><td>lspinstall/servers/html</td><td>0.018326ms</td><td>0.010974ms</td><td>0.029300ms</td></tr>
        <tr><td>lspinstall/servers/java</td><td>0.005767ms</td><td>0.013746ms</td><td>0.019513ms</td></tr>
        <tr><td>lspinstall/servers/json</td><td>0.009550ms</td><td>0.003102ms</td><td>0.012652ms</td></tr>
        <tr><td>lspinstall/servers/kotlin</td><td>0.005061ms</td><td>0.002255ms</td><td>0.007316ms</td></tr>
        <tr><td>lspinstall/servers/latex</td><td>0.009082ms</td><td>0.004108ms</td><td>0.013190ms</td></tr>
        <tr><td>lspinstall/servers/lua</td><td>0.009487ms</td><td>0.004118ms</td><td>0.013605ms</td></tr>
        <tr><td>lspinstall/servers/php</td><td>0.007664ms</td><td>0.002876ms</td><td>0.010540ms</td></tr>
        <tr><td>lspinstall/servers/puppet</td><td>0.008575ms</td><td>0.003759ms</td><td>0.012334ms</td></tr>
        <tr><td>lspinstall/servers/purescript</td><td>0.010064ms</td><td>0.003065ms</td><td>0.013129ms</td></tr>
        <tr><td>lspinstall/servers/python</td><td>0.020302ms</td><td>0.006497ms</td><td>0.026799ms</td></tr>
        <tr><td>lspinstall/servers/rome</td><td>0.008779ms</td><td>0.003493ms</td><td>0.012272ms</td></tr>
        <tr><td>lspinstall/servers/ruby</td><td>0.005688ms</td><td>0.003252ms</td><td>0.008940ms</td></tr>
        <tr><td>lspinstall/servers/rust</td><td>0.016758ms</td><td>0.007257ms</td><td>0.024015ms</td></tr>
        <tr><td>lspinstall/servers/svelte</td><td>0.024498ms</td><td>0.002475ms</td><td>0.026973ms</td></tr>
        <tr><td>lspinstall/servers/tailwindcss</td><td>0.013594ms</td><td>0.001783ms</td><td>0.015377ms</td></tr>
        <tr><td>lspinstall/servers/terraform</td><td>0.010185ms</td><td>0.009222ms</td><td>0.019407ms</td></tr>
        <tr><td>lspinstall/servers/typescript</td><td>0.011109ms</td><td>0.010233ms</td><td>0.021342ms</td></tr>
        <tr><td>lspinstall/servers/vim</td><td>0.013938ms</td><td>0.001895ms</td><td>0.015833ms</td></tr>
        <tr><td>lspinstall/servers/vue</td><td>0.036252ms</td><td>0.012379ms</td><td>0.048631ms</td></tr>
        <tr><td>lspinstall/servers/yaml</td><td>0.039218ms</td><td>0.006111ms</td><td>0.045329ms</td></tr>
        <tr><td>lspinstall/servers</td><td>0.005573ms</td><td>0.011410ms</td><td>0.016983ms</td></tr>
        <tr><td>lspinstall/util</td><td>0.004929ms</td><td>0.003910ms</td><td>0.008839ms</td></tr>
        <tr><td>lspinstall</td><td>0.009580ms</td><td>0.016533ms</td><td>0.026113ms</td></tr>
        <tr><td>lspkind</td><td>0.007582ms</td><td>0.014665ms</td><td>0.022247ms</td></tr>
        <tr><td>luasnip.config</td><td>0.006112ms</td><td>0.009967ms</td><td>0.016079ms</td></tr>
        <tr><td>luasnip.nodes.choiceNode</td><td>0.004388ms</td><td>0.013232ms</td><td>0.017620ms</td></tr>
        <tr><td>luasnip.nodes.dynamicNode</td><td>0.006275ms</td><td>0.011935ms</td><td>0.018210ms</td></tr>
        <tr><td>luasnip.nodes.functionNode</td><td>0.003997ms</td><td>0.007614ms</td><td>0.011611ms</td></tr>
        <tr><td>luasnip.nodes.insertNode</td><td>0.006467ms</td><td>0.008397ms</td><td>0.014864ms</td></tr>
        <tr><td>luasnip.nodes.node</td><td>0.006354ms</td><td>0.008454ms</td><td>0.014808ms</td></tr>
        <tr><td>luasnip.nodes.snippet</td><td>0.004618ms</td><td>0.045256ms</td><td>0.049874ms</td></tr>
        <tr><td>luasnip.nodes.textNode</td><td>0.006013ms</td><td>0.004955ms</td><td>0.010968ms</td></tr>
        <tr><td>luasnip.session</td><td>0.004829ms</td><td>0.001032ms</td><td>0.005861ms</td></tr>
        <tr><td>luasnip.util.environ</td><td>0.005613ms</td><td>0.019614ms</td><td>0.025227ms</td></tr>
        <tr><td>luasnip.util.events</td><td>0.004413ms</td><td>0.001840ms</td><td>0.006253ms</td></tr>
        <tr><td>luasnip.util.functions</td><td>0.003653ms</td><td>0.002964ms</td><td>0.006617ms</td></tr>
        <tr><td>luasnip.util.mark</td><td>0.004420ms</td><td>0.010858ms</td><td>0.015278ms</td></tr>
        <tr><td>luasnip.util.parser</td><td>0.004029ms</td><td>0.018067ms</td><td>0.022096ms</td></tr>
        <tr><td>luasnip.util.pattern_tokenizer</td><td>0.005829ms</td><td>0.008000ms</td><td>0.013829ms</td></tr>
        <tr><td>luasnip.util.types</td><td>0.003841ms</td><td>0.002937ms</td><td>0.006778ms</td></tr>
        <tr><td>luasnip.util.util</td><td>0.008949ms</td><td>0.040294ms</td><td>0.049243ms</td></tr>
        <tr><td>luasnip</td><td>0.007957ms</td><td>0.034898ms</td><td>0.042855ms</td></tr>
        <tr><td>null-ls.builtins.code-actions</td><td>0.005087ms</td><td>0.005211ms</td><td>0.010298ms</td></tr>
        <tr><td>null-ls.builtins.diagnostics</td><td>0.004175ms</td><td>0.046484ms</td><td>0.050659ms</td></tr>
        <tr><td>null-ls.builtins.formatting</td><td>0.004515ms</td><td>0.044595ms</td><td>0.049110ms</td></tr>
        <tr><td>null-ls.builtins.test</td><td>0.005663ms</td><td>0.010475ms</td><td>0.016138ms</td></tr>
        <tr><td>null-ls.builtins</td><td>0.005914ms</td><td>0.002159ms</td><td>0.008073ms</td></tr>
        <tr><td>null-ls.code-actions</td><td>0.005407ms</td><td>0.003472ms</td><td>0.008879ms</td></tr>
        <tr><td>null-ls.config</td><td>0.004485ms</td><td>0.011901ms</td><td>0.016386ms</td></tr>
        <tr><td>null-ls.diagnostics</td><td>0.005330ms</td><td>0.004171ms</td><td>0.009501ms</td></tr>
        <tr><td>null-ls.formatting</td><td>0.005703ms</td><td>0.009046ms</td><td>0.014749ms</td></tr>
        <tr><td>null-ls.generators</td><td>0.003880ms</td><td>0.011148ms</td><td>0.015028ms</td></tr>
        <tr><td>null-ls.handlers</td><td>0.003876ms</td><td>0.006634ms</td><td>0.010510ms</td></tr>
        <tr><td>null-ls.helpers</td><td>0.004119ms</td><td>0.030944ms</td><td>0.035063ms</td></tr>
        <tr><td>null-ls.info</td><td>0.005642ms</td><td>0.005478ms</td><td>0.011120ms</td></tr>
        <tr><td>null-ls.logger</td><td>0.005048ms</td><td>0.001135ms</td><td>0.006183ms</td></tr>
        <tr><td>null-ls.loop</td><td>0.005126ms</td><td>0.016733ms</td><td>0.021859ms</td></tr>
        <tr><td>null-ls.lspconfig</td><td>0.004350ms</td><td>0.008344ms</td><td>0.012694ms</td></tr>
        <tr><td>null-ls.methods</td><td>0.004890ms</td><td>0.007375ms</td><td>0.012265ms</td></tr>
        <tr><td>null-ls.rpc</td><td>0.003818ms</td><td>0.008809ms</td><td>0.012627ms</td></tr>
        <tr><td>null-ls.state</td><td>0.004303ms</td><td>0.005275ms</td><td>0.009578ms</td></tr>
        <tr><td>null-ls.utils</td><td>0.005272ms</td><td>0.014915ms</td><td>0.020187ms</td></tr>
        <tr><td>null-ls</td><td>0.006847ms</td><td>0.004841ms</td><td>0.011688ms</td></tr>
        <tr><td>nvim-treesitter-playground</td><td>0.013310ms</td><td>0.012020ms</td><td>0.025330ms</td></tr>
        <tr><td>nvim-treesitter.caching</td><td>0.004500ms</td><td>0.002893ms</td><td>0.007393ms</td></tr>
        <tr><td>nvim-treesitter.configs</td><td>0.003943ms</td><td>0.026775ms</td><td>0.030718ms</td></tr>
        <tr><td>nvim-treesitter.info</td><td>0.004554ms</td><td>0.009194ms</td><td>0.013748ms</td></tr>
        <tr><td>nvim-treesitter.install</td><td>0.006666ms</td><td>0.041414ms</td><td>0.048080ms</td></tr>
        <tr><td>nvim-treesitter.parsers</td><td>0.006082ms</td><td>0.074670ms</td><td>0.080752ms</td></tr>
        <tr><td>nvim-treesitter.query_predicates</td><td>0.003837ms</td><td>0.006564ms</td><td>0.010401ms</td></tr>
        <tr><td>nvim-treesitter.query</td><td>0.004021ms</td><td>0.016531ms</td><td>0.020552ms</td></tr>
        <tr><td>nvim-treesitter.shell_command_selectors</td><td>0.004136ms</td><td>0.018663ms</td><td>0.022799ms</td></tr>
        <tr><td>nvim-treesitter.ts_utils</td><td>0.004042ms</td><td>0.016618ms</td><td>0.020660ms</td></tr>
        <tr><td>nvim-treesitter.tsrange</td><td>0.005787ms</td><td>0.013464ms</td><td>0.019251ms</td></tr>
        <tr><td>nvim-treesitter.utils</td><td>0.006071ms</td><td>0.012117ms</td><td>0.018188ms</td></tr>
        <tr><td>nvim-treesitter</td><td>0.013138ms</td><td>0.006208ms</td><td>0.019346ms</td></tr>
        <tr><td>nvim-web-devicons</td><td>0.007025ms</td><td>0.136943ms</td><td>0.143968ms</td></tr>
        <tr><td>octo.base64</td><td>0.004971ms</td><td>0.015917ms</td><td>0.020888ms</td></tr>
        <tr><td>octo.colors</td><td>0.016638ms</td><td>0.058169ms</td><td>0.074807ms</td></tr>
        <tr><td>octo.completion</td><td>0.004205ms</td><td>0.006913ms</td><td>0.011118ms</td></tr>
        <tr><td>octo.config</td><td>0.004148ms</td><td>0.048353ms</td><td>0.052501ms</td></tr>
        <tr><td>octo.constants</td><td>0.004177ms</td><td>0.012538ms</td><td>0.016715ms</td></tr>
        <tr><td>octo.date</td><td>0.004278ms</td><td>0.109407ms</td><td>0.113685ms</td></tr>
        <tr><td>octo.folds</td><td>0.007726ms</td><td>0.004374ms</td><td>0.012100ms</td></tr>
        <tr><td>octo.gh</td><td>0.004740ms</td><td>0.022470ms</td><td>0.027210ms</td></tr>
        <tr><td>octo.graphql</td><td>0.007422ms</td><td>0.071050ms</td><td>0.078472ms</td></tr>
        <tr><td>octo.mappings</td><td>0.004420ms</td><td>0.024827ms</td><td>0.029247ms</td></tr>
        <tr><td>octo.model.body-metadata</td><td>0.006160ms</td><td>0.002217ms</td><td>0.008377ms</td></tr>
        <tr><td>octo.model.octo-buffer</td><td>0.005888ms</td><td>0.078169ms</td><td>0.084057ms</td></tr>
        <tr><td>octo.model.thread-metadata</td><td>0.004744ms</td><td>0.003236ms</td><td>0.007980ms</td></tr>
        <tr><td>octo.model.title-metadata</td><td>0.004540ms</td><td>0.001220ms</td><td>0.005760ms</td></tr>
        <tr><td>octo.reviews.file-entry</td><td>0.004651ms</td><td>0.027793ms</td><td>0.032444ms</td></tr>
        <tr><td>octo.reviews.file-panel</td><td>0.004417ms</td><td>0.035242ms</td><td>0.039659ms</td></tr>
        <tr><td>octo.reviews.layout</td><td>0.004348ms</td><td>0.023523ms</td><td>0.027871ms</td></tr>
        <tr><td>octo.reviews.renderer</td><td>0.006058ms</td><td>0.007780ms</td><td>0.013838ms</td></tr>
        <tr><td>octo.reviews</td><td>0.005864ms</td><td>0.048704ms</td><td>0.054568ms</td></tr>
        <tr><td>octo.signs</td><td>0.005635ms</td><td>0.007950ms</td><td>0.013585ms</td></tr>
        <tr><td>octo.ui.bubbles</td><td>0.004795ms</td><td>0.006497ms</td><td>0.011292ms</td></tr>
        <tr><td>octo.utils</td><td>0.003701ms</td><td>0.098548ms</td><td>0.102249ms</td></tr>
        <tr><td>octo.window</td><td>0.003896ms</td><td>0.019415ms</td><td>0.023311ms</td></tr>
        <tr><td>octo.writers</td><td>0.004535ms</td><td>0.123920ms</td><td>0.128455ms</td></tr>
        <tr><td>octo</td><td>0.011948ms</td><td>0.030383ms</td><td>0.042331ms</td></tr>
        <tr><td>packer.load</td><td>0.034542ms</td><td>0.024793ms</td><td>0.059335ms</td></tr>
        <tr><td>plenary.async.async</td><td>0.005965ms</td><td>0.006824ms</td><td>0.012789ms</td></tr>
        <tr><td>plenary.async.control</td><td>0.004094ms</td><td>0.025538ms</td><td>0.029632ms</td></tr>
        <tr><td>plenary.async.structs</td><td>0.004656ms</td><td>0.003900ms</td><td>0.008556ms</td></tr>
        <tr><td>plenary.async.util</td><td>0.004252ms</td><td>0.006435ms</td><td>0.010687ms</td></tr>
        <tr><td>plenary.async_lib.api</td><td>0.003997ms</td><td>0.001604ms</td><td>0.005601ms</td></tr>
        <tr><td>plenary.async_lib.async</td><td>0.004029ms</td><td>0.012264ms</td><td>0.016293ms</td></tr>
        <tr><td>plenary.async_lib.lsp</td><td>0.003941ms</td><td>0.001287ms</td><td>0.005228ms</td></tr>
        <tr><td>plenary.async_lib.structs</td><td>0.005848ms</td><td>0.003081ms</td><td>0.008929ms</td></tr>
        <tr><td>plenary.async_lib.tests</td><td>0.005642ms</td><td>0.001592ms</td><td>0.007234ms</td></tr>
        <tr><td>plenary.async_lib.util</td><td>0.005954ms</td><td>0.016970ms</td><td>0.022924ms</td></tr>
        <tr><td>plenary.async_lib.uv_async</td><td>0.005799ms</td><td>0.003346ms</td><td>0.009145ms</td></tr>
        <tr><td>plenary.async_lib</td><td>0.005057ms</td><td>0.003300ms</td><td>0.008357ms</td></tr>
        <tr><td>plenary.bit</td><td>0.005886ms</td><td>0.012796ms</td><td>0.018682ms</td></tr>
        <tr><td>plenary.errors</td><td>0.004654ms</td><td>0.002240ms</td><td>0.006894ms</td></tr>
        <tr><td>plenary.functional</td><td>0.003840ms</td><td>0.004194ms</td><td>0.008034ms</td></tr>
        <tr><td>plenary.job</td><td>0.005120ms</td><td>0.026045ms</td><td>0.031165ms</td></tr>
        <tr><td>plenary.log</td><td>0.005721ms</td><td>0.024572ms</td><td>0.030293ms</td></tr>
        <tr><td>plenary.path</td><td>0.004852ms</td><td>0.041434ms</td><td>0.046286ms</td></tr>
        <tr><td>plenary.strings</td><td>0.005191ms</td><td>0.008466ms</td><td>0.013657ms</td></tr>
        <tr><td>plenary.tbl</td><td>0.003993ms</td><td>0.002378ms</td><td>0.006371ms</td></tr>
        <tr><td>plenary.vararg.rotate</td><td>0.004192ms</td><td>0.004013ms</td><td>0.008205ms</td></tr>
        <tr><td>plenary.vararg</td><td>0.004718ms</td><td>0.001025ms</td><td>0.005743ms</td></tr>
        <tr><td>spaceless</td><td>0.005444ms</td><td>0.011508ms</td><td>0.016952ms</td></tr>
        <tr><td>spellsitter</td><td>0.014532ms</td><td>0.017292ms</td><td>0.031824ms</td></tr>
        <tr><td>symbols-outline.config</td><td>0.006380ms</td><td>0.020545ms</td><td>0.026925ms</td></tr>
        <tr><td>symbols-outline.markdown</td><td>0.003768ms</td><td>0.009232ms</td><td>0.013000ms</td></tr>
        <tr><td>symbols-outline.parser</td><td>0.005913ms</td><td>0.010390ms</td><td>0.016303ms</td></tr>
        <tr><td>symbols-outline.symbols</td><td>0.004973ms</td><td>0.003857ms</td><td>0.008830ms</td></tr>
        <tr><td>symbols-outline.ui</td><td>0.004893ms</td><td>0.008726ms</td><td>0.013619ms</td></tr>
        <tr><td>symbols-outline.utils.init</td><td>0.003801ms</td><td>0.004344ms</td><td>0.008145ms</td></tr>
        <tr><td>symbols-outline.utils.lsp_utils</td><td>0.004786ms</td><td>0.003401ms</td><td>0.008187ms</td></tr>
        <tr><td>symbols-outline.view</td><td>0.004395ms</td><td>0.004980ms</td><td>0.009375ms</td></tr>
        <tr><td>symbols-outline.writer</td><td>0.004174ms</td><td>0.005715ms</td><td>0.009889ms</td></tr>
        <tr><td>symbols-outline</td><td>0.011396ms</td><td>0.020787ms</td><td>0.032183ms</td></tr>
        <tr><td>telescope._extensions.fzf</td><td>0.013293ms</td><td>0.013910ms</td><td>0.027203ms</td></tr>
        <tr><td>telescope._extensions</td><td>0.005613ms</td><td>0.003299ms</td><td>0.008912ms</td></tr>
        <tr><td>telescope.config</td><td>0.005340ms</td><td>0.032133ms</td><td>0.037473ms</td></tr>
        <tr><td>telescope.deprecated</td><td>0.006162ms</td><td>0.006181ms</td><td>0.012343ms</td></tr>
        <tr><td>telescope.log</td><td>0.003956ms</td><td>0.002155ms</td><td>0.006111ms</td></tr>
        <tr><td>telescope.sorters</td><td>0.005705ms</td><td>0.030787ms</td><td>0.036492ms</td></tr>
        <tr><td>telescope.utils</td><td>0.006294ms</td><td>0.033068ms</td><td>0.039362ms</td></tr>
        <tr><td>telescope</td><td>0.007769ms</td><td>0.006470ms</td><td>0.014239ms</td></tr>
        <tr><td>treesitter-context.utils</td><td>0.008784ms</td><td>0.002382ms</td><td>0.011166ms</td></tr>
        <tr><td>treesitter-context</td><td>0.012644ms</td><td>0.038443ms</td><td>0.051087ms</td></tr>
        <tr><td>trouble.colors</td><td>0.004418ms</td><td>0.007903ms</td><td>0.012321ms</td></tr>
        <tr><td>trouble.config</td><td>0.003711ms</td><td>0.011753ms</td><td>0.015464ms</td></tr>
        <tr><td>trouble.folds</td><td>0.003696ms</td><td>0.004821ms</td><td>0.008517ms</td></tr>
        <tr><td>trouble.providers.lsp</td><td>0.006261ms</td><td>0.007559ms</td><td>0.013820ms</td></tr>
        <tr><td>trouble.providers.qf</td><td>0.005869ms</td><td>0.004737ms</td><td>0.010606ms</td></tr>
        <tr><td>trouble.providers.telescope</td><td>0.019361ms</td><td>0.007350ms</td><td>0.026711ms</td></tr>
        <tr><td>trouble.providers</td><td>0.004138ms</td><td>0.005340ms</td><td>0.009478ms</td></tr>
        <tr><td>trouble.renderer</td><td>0.004471ms</td><td>0.016762ms</td><td>0.021233ms</td></tr>
        <tr><td>trouble.text</td><td>0.004188ms</td><td>0.004188ms</td><td>0.008376ms</td></tr>
        <tr><td>trouble.util</td><td>0.003852ms</td><td>0.012615ms</td><td>0.016467ms</td></tr>
        <tr><td>trouble.view</td><td>0.004475ms</td><td>0.040955ms</td><td>0.045430ms</td></tr>
        <tr><td>trouble</td><td>0.008870ms</td><td>0.020524ms</td><td>0.029394ms</td></tr>
        <tr><td>vim.F</td><td>0.005481ms</td><td>0.001713ms</td><td>0.007194ms</td></tr>
        <tr><td>vim.highlight</td><td>0.006384ms</td><td>0.023151ms</td><td>0.029535ms</td></tr>
        <tr><td>vim.lsp.buf</td><td>0.006028ms</td><td>0.028343ms</td><td>0.034371ms</td></tr>
        <tr><td>vim.lsp.codelens</td><td>0.005359ms</td><td>0.015432ms</td><td>0.020791ms</td></tr>
        <tr><td>vim.lsp.diagnostic</td><td>0.008004ms</td><td>0.064811ms</td><td>0.072815ms</td></tr>
        <tr><td>vim.lsp.handlers</td><td>0.006128ms</td><td>0.028319ms</td><td>0.034447ms</td></tr>
        <tr><td>vim.lsp.log</td><td>0.025553ms</td><td>0.009086ms</td><td>0.034639ms</td></tr>
        <tr><td>vim.lsp.protocol</td><td>0.021170ms</td><td>0.059855ms</td><td>0.081025ms</td></tr>
        <tr><td>vim.lsp.rpc</td><td>0.007450ms</td><td>0.041752ms</td><td>0.049202ms</td></tr>
        <tr><td>vim.lsp.util</td><td>0.007334ms</td><td>0.119266ms</td><td>0.126600ms</td></tr>
        <tr><td>vim.lsp</td><td>0.003642ms</td><td>0.084496ms</td><td>0.088138ms</td></tr>
        <tr><td>vim.treesitter.highlighter</td><td>0.006430ms</td><td>0.025854ms</td><td>0.032284ms</td></tr>
        <tr><td>vim.treesitter.language</td><td>0.005506ms</td><td>0.002717ms</td><td>0.008223ms</td></tr>
        <tr><td>vim.treesitter.languagetree</td><td>0.005822ms</td><td>0.023099ms</td><td>0.028921ms</td></tr>
        <tr><td>vim.treesitter.query</td><td>0.005773ms</td><td>0.020355ms</td><td>0.026128ms</td></tr>
        <tr><td>vim.treesitter</td><td>0.003865ms</td><td>0.006707ms</td><td>0.010572ms</td></tr>
        <tr><td>vim.uri</td><td>0.004444ms</td><td>0.008966ms</td><td>0.013410ms</td></tr>
    </tbody>
</table>
</details>

Cache load: 3.414473ms

Total resolve: 2.122016ms, total load: 4.539859ms

## Relevent Neovim PR's

[libs: vendor libmpack and libmpack-lua](https://github.com/neovim/neovim/pull/15566) [merged]

[fix(runtime): add compressed representation to &rtp](https://github.com/neovim/neovim/pull/15867) [merged]

[fix(runtime): don't use regexes inside lua require'mod'](https://github.com/neovim/neovim/pull/15973) [merged]

[feat(lua): startup profiling](https://github.com/neovim/neovim/pull/15436)


## Credit

All credit goes to @bfredl who implemented the majority of this plugin in https://github.com/neovim/neovim/pull/15436.
