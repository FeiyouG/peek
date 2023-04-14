# Design

## Peek

### Configuration

- Configuration file can be found at
  `$XDG_CONFIG_HOME/peek/int.lua`
- Configuration file needs to return a `lua` table
  with the following fields:
  - `adaptors` (list, default is empty)
    - Each element contains the following fields:
      - `[1]`: 
        (required) 
        the GitHub link of the adaptor
      - `config`: 
        (optional)
        the configuration of that adaptor
  - `sync_config` (`boolean`, default is `true`)
    - If set to true, 
      then `peek` will automatically update configuration file
      when invoked on a command line
      (e.g. `peek adaptor install google`
      will install the Google adaptor
      and add a corresponding entry to `adaptor` in the config file)

### Commands

- `peek adaptor`
  - `install [adaptor-name]`
  - `remove [adaptor-name]`
  - `update [adaptor-name]`
  - Adaptors are installed in 
    `$XDG_DATA_HOME/peek/adaptors`.
- `peek [adaptor-name]`
  - `fetch [api] [kwards]`: fetch a webpage directly
  - flags:
    - `to`: the output format
      - `markdown`: 
        (default)

### Exposed APIs

- `peek.parser`
  - `parse_html`
  - `parse_json`
- `peek.async`
  - `fetch`:
- `peek.page.component`
  - `header`
    - parameters: 
      - level: `int`
      - text: `string`
  - `bold`
  - `italic`
  - `break`
  - `list`
    - parameters:
      - ordered: `boolean`
      - elements: list of `components`
  - `link`
    - parameters:
      - link: `string`
      - text: `string`
  - `...`

## Adaptor

Each adaptor needs to return a `lua` table
with the following entries:
- `capabilities` (table)
- `fetch` (function)


## An Example Flow
```
        peek                                            adaptor
                       ┌───────────────────┐
                       │peek google neovim │
                       └─────────┬─────────┘
             ┌───────────────────┘
             ▼
  ┌──────────────────────────┐
  │ peek checks wehter       │
  │ google adaptor exists,   │
  │ then invokes             ├───────────────────────────┐
  │ `google.fetch("neovim")` │                           │
  └──────────────────────────┘                           ▼
                                            ┌────────────────────────────┐
                                            │Parses the input into an api│ 
                                            │and calls `peek.async.fetch`│ 
                                            └─────────────┬──────────────┘
                                                          ▼ 
                                           ┌───────────────────────────────┐
                                           │Parses the returned json string│
                                           │into an abstract syntax tree   │
                                           │with `peek.parser.parse_json`  │
                                           └──────────────┬────────────────┘
                                                          ▼ 
                                           ┌─────────────────────────────────┐  
                                           │Construct and return a lua table │                                                          
                                           └──────────────┬──────────────────┘  
            ┌─────────────────────────────────────────────┘
            ▼
   ┌──────────────────────┐
   │peek parses the table │
   │into the specified    │
   │output format         │
   └──────────────────────┘
```
