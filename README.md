`reason-mode`: A major Emacs mode for Reason syntax
============================================================

At the moment this plugin is not deployed on Elpa. The sections below explains how to manually install it.
Alternatively, you can use [quelpa](https://github.com/quelpa/quelpa) and the following recipe:

```lisp
(quelpa '(reason-mode :repo "reasonml-editor/reason-mode" :fetcher github :stable t))
```

### Manual Installation

Download `reason-indent.el`, `reason-interaction.el`, `reason-mode.el` and `refmt.el` at the root of this repository and place it in a `vendor` file next to your Emacs configuration files. Then place the following somewhere in you `.emacs.el`:

```lisp
(add-to-list 'load-path "/path/to/vendor")
```

**Note**: the following setup assumes Reason and Merlin are installed. This can be achieved by either installing `reason-cli` from NPM (`npm -g install git://github.com/reasonml/reason-cli.git`), which ships all executables needed to run Reason, or from OPAM (`opam install reason.1.13.7 merlin`).

Add the following to your `~/.emacs` or `~/.emacs.d/init.el` file:

```lisp
;;----------------------------------------------------------------------------
;; Reason setup
;;----------------------------------------------------------------------------

(defun chomp-end (str)
  "Chomp tailing whitespace from STR."
  (replace-regexp-in-string (rx (* (any " \t\n")) eos)
                            ""
                            str))

(defun shell-cmd (cmd)
  "Returns the stdout output of a shell command or nil if the command returned
   an error"
  (let ((stdoutput (chomp-end
                    (with-output-to-string
                      (with-current-buffer
                          standard-output
                        (process-file shell-file-name nil
                                      '(t nil)  nil
                                      shell-command-switch cmd))))))
    (when (not (= (length stdoutput) 0))
      stdoutput)))

(let* ((refmt-bin (or (shell-cmd "refmt ----where")
                      (shell-cmd "which refmt")))
       (merlin-bin (or (shell-cmd "ocamlmerlin ----where")
                       (shell-cmd "which ocamlmerlin")))
       (merlin-base-dir (when merlin-bin
                          (replace-regexp-in-string "bin/ocamlmerlin$" "" merlin-bin))))
  ;; Add npm merlin.el to the emacs load path and tell emacs where to find ocamlmerlin
  (when merlin-bin
    (add-to-list 'load-path (concat merlin-base-dir "share/emacs/site-lisp/"))
    (setq merlin-command merlin-bin))

  (when refmt-bin
    (setq refmt-command refmt-bin)))

(require 'reason-mode)
(require 'merlin)
(add-hook 'reason-mode-hook (lambda ()
                              (add-hook 'before-save-hook 'refmt-before-save)
                              (merlin-mode)))

(setq merlin-ac-setup t)
```

If you have iedit mode set up: 

```lisp
(require 'merlin-iedit)
(defun evil-custom-merlin-iedit ()
  (interactive)
  (if iedit-mode (iedit-mode)
    (merlin-iedit-occurrences)))
(define-key merlin-mode-map (kbd "C-c C-e") 'evil-custom-merlin-iedit)
```

(Thanks @sgrove: [https://gist.github.com/sgrove/c9bdfed77f4da8db108dfb2c188f7baf](https://gist.github.com/sgrove/c9bdfed77f4da8db108dfb2c188f7baf))

This associates `reason-mode` with `.re` and `.rei` files. To enable it explicitly, do <kbd>M-x reason-mode</kbd>.

### Utop

Reason-mode provides (opt-in) `rtop` support. At the moment only the native workflow is supported.

First of all you need to install the [Utop Emacs integration](https://github.com/diml/utop#integration-with-emacs). Make sure it is latest `master` because the feature is fairly new.

Then in your Emacs init file add:

```lisp
(require 'utop)
(setq utop-command "opam config exec -- rtop -emacs")
(add-hook 'reason-mode-hook #'utop-minor-mode) ;; can be included in the hook above as well
```

After this, the function `utop` (`C-c C-s`) will start `rtop` in Reason buffers.

### Spacemacs

There is currently no offical reason layer available, but you can install the `reason-mode` package automatically.
Some are working on a layer in the meantime [#1149](https://github.com/facebook/reason/issues/1149). 

```lisp
dotspacemacs-additional-packages
  '(
    (reason-mode
      :location (recipe
        :repo "reasonml-editor/reason-mode"
        :fetcher github
        :files ("reason-mode.el" "refmt.el")))
)
```

Afterwards add the [snippet](#manual-installation) to your `dotspacemacs/user-config`.

### Features

#### Auto-format before saving

If you have refmt installed, you can add this to your `.emacs` file to enable
auto-format:

```lisp
(add-hook 'reason-mode-hook (lambda ()
          (add-hook 'before-save-hook 'refmt-before-save)))
```

### Tests via ERT

The `test` folder contains tests that can be run via [ERT](http://www.gnu.org/software/emacs/manual/html_node/ert/index.html).  You
can use `run_reason_emacs_tests.sh` to run them in batch mode, if you set the environment variable EMACS to a program that runs emacs.

To test it under emacs 23, which does not ship with ERT, download ert.el from
https://raw.githubusercontent.com/ohler/ert/c619b56c5bc6a866e33787489545b87d79973205/lisp/emacs-lisp/ert.el
and put it in a place where emacs can find it.  (ERT has shipped with emacs
since emacs 24.)

## License

`reason-mode` is distributed under the terms of both the MIT license and the
Apache License (Version 2.0).

See [LICENSE-MIT](LICENSE-MIT) and [LICENSE-APACHE](LICENSE-APACHE) for details.
