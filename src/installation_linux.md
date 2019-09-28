# Installing Rust with Rustup

- Visit [rustup.rs](https://rustup.rs), to install the rust toolchain.

# Choosing your editor
Whatever editor you choose, the goal is for you to have RLS and Code Formating (rustfmt) installed and enabled. VSCode is a mature and populare solution for Rust, but Vim is usable as well.

## Configuring vim 8.x for Rust with the RLS
The steps below configure vim 8.x to support Rust and use the Rust Language Server (RLS) for autocompletion.

1. Make sure you have a working Git installation
2. Visit [https://rls.booyaa.wtf](https://rls.booyaa.wtf) and follow the steps for your preferred Vim package management strategy.
3. Modify your vimrc to include the below snippet. The guide linked in the previous step is still configured to use the Rust nightly build, since the RLS used to only be available in the nightly builds. RLS is now available in stable, and we installed stable Rust (the default). The below snippet should replace the one from the linked guide, and changes the 'cmd' to use stable instead of nightly.
   
   ```text
    if executable('rls')
        au User lsp_setup call lsp#register_server({
            \ 'name': 'rls',
            \ 'cmd': {server_info->['rustup', 'run', 'stable', 'rls']},
            \ 'whitelist': ['rust'],
            \ })
    endif
   ```
4. Make sure `filetype plugin indent on` and `syntax enable` and lets add format on save as well like `let g:rustfmt_autosave = 1`
5. Restart vim or reload your vimrc
6. Open a Rust file and test our autocompletion (for example start typing `use std::`)

    ![vim Rust autocompletion](./images/install/gvimrls.png)

## Configuring VS Code for Rust with the RLS
The steps below configure VS Code on Mac to support Rust and use the Rust Language Server (RLS) for autocompletion and incremental compilation to display warnings and errors.

1. Install the Rust (rls) extension by user 'rust-lang' in VS Code. There are several other plugins, but this one is the most maintained.
2. Reload the window in VS Code, or restart VS Code
3. If you see an error message that the RLS could not be started or that the extension could not find rustup, then you will have to configure VS Code's path for rustup:
   1. Open VS Code preferences and navigate to the Rust extension preferences
   2. Modify the rustup path to use an absolute path to your installation: `C:\Users\<username>\.cargo\bin\rustup`

        ![VS Code rustup path](./images/install/vscode_rust.png)

   3. Reload the window in VS Code, or restart VS Code
   4. You may see a prompt in the lower-right to install the RLS. If so, click yes.

4. Open a Rust file and test out the RLS:
   1. Try autocompletion (for example start typing `use std::`) at the top of a file
   2. Try the incremental compilation (for example `println!("Hello, world!") blah blah 42 42` should show an inline error)
   
    ![VS Code Rust autocompletion and incremental compilation](./images/install/vscode_rust2.png)

5.  Enable format on save in VScode settings. Code->Preferences->Settings->Text Editor->Formatting->Format On Save
