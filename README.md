# persp-mode  

## Intro  
Perspectives for emacs, based on [`perspective-el`](http://github.com/nex3/perspective-el) by Nathan Weizenbaum.  
But perspectives shared among frames \+ ability to save/restore from/to file.  

## Installation  
persp-mode is available from [`MELPA`](https://github.com/milkypostman/melpa). So if you use this repo installation is easy:  
`M-x: package-install RET persp-mode RET`  
Alternatively you can download persp-mode.el from [`github`](https://github.com/Bad-ptr/persp-mode.el) and install it with:  
`M-x: package-install-file RET 'path_to_where_you_saved_persp-mode.el' RET`  

Another(oldschool;p) way:  
Put persp-mode.el file somewhere in your emacs load-path.  

### Suggested configuration  
It's recommended to switch off animation on restoring window configuration with `workgroups.el`.  
(it's clashing with `golden-ration-mode` for example, sometimes erring when creating new frame
and it's slow on remote network connections.)  
You can do it with: `(setq wg-morph-on nil)`.  

#### When installing from MELPA  
```lisp
(with-eval-after-load "persp-mode-autoloads"
  (setq wg-morph-on nil) ;; switch off animation of restoring window configuration
  (add-hook 'after-init-hook #'(lambda () (persp-mode 1))))
```
#### When installing without generation of autoloads  
```lisp
(with-eval-after-load "persp-mode"
  (setq wg-morph-on nil)
  (add-hook 'after-init-hook #'(lambda () (persp-mode 1))))
(require 'persp-mode)
```

### Dependencies  
Ability of saving/restoring window configurations from/to file depends on [`workgroups.el`](https://github.com/tlh/workgroups.el).  
It's automatically installed if you install persp-mode from mepla, otherwise you must download it and put somewhere in your emacs load-path.  

## Keys  
`C-x x s` -- create/switch to perspective.  
`C-x x r` -- rename perspective.  
`C-x x c` -- kill perspective. (if you try to kill 'none' persp -- it'l kill all opened buffers).  
`C-x x a` -- add buffer to perspective.  
`C-x x t` -- switch to buffer without adding it to current perspective.  
`C-x x i` -- import all buffers from another perspective.  
`C-x x k` -- remove buffer from perspective.  
`C-x x w` -- save perspectives to file.  
`C-x x l` -- load perspectives from file.  

## Customization  
`M-x: customize-group RET persp-mode RET`  


## Interaction with side packages  

### Speedbar  
```lisp
(add-to-list 'speedbar-frame-parameters (cons 'persp-ignore-wconf t))
```

### Ibuffer  
[gist](https://gist.github.com/Bad-ptr/7644606)

---

## Troubles  
When you create new frame(with `emacsclient -c` for example)
the selected window of created frame is switching to `*scratch*` buffer. This behaviour fixed in emacs version >= 24.4(and in current emacs trunk).
Alternatively you can save `server.el` from `/usr/share/emacs/${your_emacs_version_number}/lisp/`
(or from source tree, or from somewhere else) to directory in your `load-path` and edit it like that(this works for emacs 24.3 at least):  
replace  
```lisp
(unless (or files commands)
  (if (stringp initial-buffer-choice)
      (find-file initial-buffer-choice)
    (switch-to-buffer (get-buffer-create "*scratch*")
                      'norecord)))
```

by  

```lisp
(unless (or files commands)
  (let ((buf
         (cond ((stringp initial-buffer-choice)
                (find-file-noselect initial-buffer-choice))
               ((functionp initial-buffer-choice)
                (funcall initial-buffer-choice)))))
    (switch-to-buffer
     (if (buffer-live-p buf) buf (get-buffer-create "*scratch*"))
     'norecord)))
```
and set variable `persp-is-ibc-as-f-supported` to `t`.  


Or you can edit persp-mode.el. Find `defun* persp-activate`, replace  
```lisp
(persp-restore-window-conf frame persp new-frame)
```
with something like that:  
```lisp
(lexical-let ((frm frame)
              (prsp persp)
              (n-frm new-frame))
    (run-at-time 0.1 nil
       #'(lambda () (persp-restore-window-conf frm prsp n-frm))))
```
Then find `defun* persp-restore-window-conf` And remove this part:  
```lisp
(when persp-is-ibc-as-f-supported
                (if new-frame
                    (lexical-let ((cbuf (current-buffer)))
                      (setq initial-buffer-choice
                            #'(lambda () (setq initial-buffer-choice nil) cbuf)))
                  (when (functionp initial-buffer-choice)
                    (switch-to-buffer (funcall initial-buffer-choice)))))
```
