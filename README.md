# Tangonov

[![img](https://melpa.org/packages/tangonov-theme-badge.svg)](https://melpa.org/#/tangonov-theme)

![img](images/tangonov.png)

A pleasant dark theme with bright, easy to read pastels. I had originally made this theme as a contribution to Doom Themes as "doom-material-dark". This is the stand-alone verison of that theme.


![img](images/tangonov-1.png) ![img](images/tangonov-2.png)

1.  [Package Header](#orgf3cf1fa)
2.  [Dependences](#org6ab210f)
3.  [Utility Functions](#orgcaa5bc2)
    1.  [Converting named colors to hexidecimal colors](#org12e85f8)
    2.  [Blending colors](#org823d48d)
4.  [Custom Group](#org226ecbd)
5.  [Custom Fringes](#org1b8e81a)
    1.  [A Smaller Fringe by Default](#org3a4641e)
    2.  [Bitmaps](#org18b741b)
    3.  [Bookmark.el](#org2f2a833)
    4.  [Flycheck](#org72cd2d5)
    5.  [Flymake](#orgf230fa3)
    6.  [Apply Settings](#orga2b841c)
6.  [Color Definitions](#org5e8f13e)
7.  [Faces](#orgfe6e824)
    1.  [Ansi Colors](#orgc43f92c)
    2.  [Avy](#org833c2f5)
    3.  [Company](#org68e597c)
    4.  [CSS](#orgc90cceb)
    5.  [Easy Customization](#orgb448647)
    6.  [Elfleed](#org01851d6)
    7.  [EWW](#org8e9c7f0)
    8.  [Emacs](#orge443435)
    9.  [Email](#org8fee7a6)
    10. [ERC](#orgea1a24a)
    11. [Evil](#org17ffb0f)
    12. [Font Lock Faces](#org445868f)
    13. [Goggles](#orgdbf3613)
    14. [Hydra](#orgd01a68e)
    15. [Inf-Ruby](#orgc609bbb)
    16. [ISearch](#org4823504)
    17. [Keycast](#orgd507d67)
    18. [Linters](#orgac01855)
    19. [LSP](#org741a765)
    20. [Mode-line](#org7523082)
    21. [Org Mode](#orga01dc3d)
    22. [Rainbow Delimiters](#org2312c4a)
    23. [RJSX Mode](#org2bb0704)
    24. [Shells](#orgdb6353e)
    25. [Tab Bar/Line](#org357c097)
    26. [Typescript.el](#org1823062)
    27. [Version Control](#org759dfcc)
    28. [Web Mode](#org67b531a)
    29. [Widgets](#org32ddadf)
    30. [End of Custom Faces](#org9f0855b)
8.  [Add Theme to Custom Load Path](#orgdb4c000)
9.  [Provide the Theme](#orgd721327)
10. [Theme Footer](#orgebea302)
11. [Contributing](#org2eda3e0)
12. [License](#org46e2ab2)


<a id="orgf3cf1fa"></a>

## Package Header

Some preliminary information about the theme

```elisp
;;; tangonov-theme.el --- A 256 color dark theme featuring bright pastels -*- lexical-binding: t -*-

;; Copyright (C) 2022 Trevor Richards

;; Author: Trevor Richards <trev@trevdev.ca>
;; Maintainer: Trevor Richards <trev@trevdev.ca>
;; URL: https://github.com/trev-dev/tangonov-theme
;; Created: 20th July, 2022
;; Keywords: faces, theme, dark, fringe
;; Version: 1.3.1
;; Package-Requires: ((emacs "25"))

;; License: GPL3

;; This program is free software; you can redistribute it and/or modify
;; it under the terms of the GNU General Public License as published by
;; the Free Software Foundation, either version 3 of the License, or
;; (at your option) any later version.

;; but WITHOUT ANY WARRANTY; without even the implied warranty of
;; This program is distributed in the hope that it will be useful,
;; MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
;; GNU General Public License for more details.

;; You should have received a copy of the GNU General Public License
;; along with this program.  If not, see <http://www.gnu.org/licenses/>.

;;; Commentary:

;; Somewhat inspired by Material Dark, Tangonov aims to be a dark theme with
;; bright, pleasant pastel colors that are easy to distinguish from one another.

;;; Code:
;; Note: This file was generated using literate programming. See tangonov-theme.org.
```


<a id="org6ab210f"></a>

## Dependences

We will need a relatively modern version of emacs with "Common Lisp" support. These are for [utility functions](#orgcaa5bc2).

```elisp
(require 'cl-lib)
```


<a id="orgcaa5bc2"></a>

## Utility Functions

[Doom Themes](https://github.com/doomemacs/themes) has some really helpful color functions that already implement the math for blending/calculating RGB colors for me so that I don't have to do that myself.

The benefit of using these is laziness. I can make one list of colors and then derive other complimentary colors from that list.


<a id="org12e85f8"></a>

### Converting named colors to hexidecimal colors

Apparently Emacs has a built in function called `tty-color-standard-values` which gives us the ability to derive an RGB value from any "named color" such as "red," "blue," etc.

```elisp
(defun tangonov--get-rgb (color)
  "Get the hexidecimal version of the named `COLOR'."
  (cl-loop with div = (float (car (tty-color-standard-values "#ffffff")))
           for x in (tty-color-standard-values (downcase color))
           collect (/ x div)))
```


<a id="org823d48d"></a>

### Blending colors

With `color-blend` we can blend any two arbitrary hexidecimal colors with this helper function by a coefficient of an alpha between 0.0-1.0.

```elisp
(defun tangonov-blend (c1 c2 alpha)
  "Blend hexidecimal colors `C1' and `C2' together by a coefficient of `ALPHA'."
  (when (and c1 c2)
    (cond ((or (listp c1) (listp c2))
           (cl-loop for x in c1
                    when (if (listp c2) (pop c2) c2)
                    collect (tangonov-blend x it alpha)))
          ((and (string-prefix-p "#" c1) (string-prefix-p "#" c2))
           (apply (lambda (r g b)
                    (format "#%02x%02x%02x" (* r 255) (* g 255) (* b 255)))
                  (cl-loop for it    in (tangonov--get-rgb c1)
                           for other in (tangonov--get-rgb c2)
                           collect (+ (* alpha it) (* other (- 1 alpha))))))
          (c1))))
```

Here we has some quick derivations of `color-blend` which can quickly darken, or lighten a color.

```elisp
(defun tangonov-darken (color alpha)
  "Darken a hexidecimal `COLOR' by a coefficient of `ALPHA'.
Alpha should be a float between 0 and 1."
  (cond ((listp color)
         (cl-loop for c in color collect (tangonov-darken c alpha)))
        ((tangonov-blend color "#000000" (- 1 alpha)))))

(defun tangonov-lighten (color alpha)
  "Lighten a hexidecimal `COLOR' by a coefficient of `ALPHA'.
Alpha should be a float between 0 and 1."
  (cond ((listp color)
         (cl-loop for c in color collect (tangonov-lighten c alpha)))
        ((tangonov-blend color "#FFFFFF" (- 1 alpha)))))
```


<a id="org226ecbd"></a>

## Custom Group

This theme will be customizable under the group name `tangonov-theme`

```elisp
(defgroup tangonov-theme nil
  "Custom settings for `tangonov-theme'.")
```


<a id="org1b8e81a"></a>

## Custom Fringes

This is an experimental feature that includes opinionated settings for a custom fringe.

```elisp
(defcustom tangonov-enable-custom-fringes t
  "Use custom settings from tangonov-theme to set up your fringe area."
  :type 'boolean
  :group 'tangonov-theme)
```


<a id="org3a4641e"></a>

### A Smaller Fringe by Default

```elisp
(fringe-mode '(4 . 0))
```


<a id="org18b741b"></a>

### Bitmaps

With the use of `(define-fringe-bitmap)` we can set custom bitmaps to be used by built-ins and packages. They key to making these work is to load them after the packages that define theme. They are not "customizable", so the must be overridden.

1.  Triangle Bitmap

    ```elisp
    (defvar tangonov--fringe-right-triangle
      (vector #b00000000
              #b00000000
              #b00000000
              #b00000000
              #b00000000
              #b10000000
              #b11000000
              #b11100000
              #b11110000
              #b11100000
              #b11000000
              #b10000000
              #b00000000
              #b00000000
              #b00000000
              #b00000000
              #b00000000)
      "A fringe bitmap used by tangonov-theme.")
    ```

2.  Stub Bitmap

    ```elisp
    (defvar tangonov--fringe-stub
      (vector #b00000000
              #b00000000
              #b00000000
              #b00000000
              #b00000000
              #b11110000
              #b11110000
              #b11110000
              #b11110000
              #b11110000
              #b11110000
              #b11110000
              #b00000000
              #b00000000
              #b00000000
              #b00000000
              #b00000000)
      "A fringe bitmap used by tangonov-theme.")
    ```


<a id="org2f2a833"></a>

### Bookmark.el

```elisp
(with-eval-after-load 'bookmark
  (define-fringe-bitmap 'bookmark-fringe-mark
    tangonov--fringe-stub))
```


<a id="org72cd2d5"></a>

### Flycheck

```elisp
(with-eval-after-load 'flycheck
  (define-fringe-bitmap 'flycheck-fringe-bitmap-caret
    tangonov--fringe-right-triangle)
  (flycheck-define-error-level
        'error
      :severity 100
      :compilation-level 2
      :overlay-category 'flycheck-error-overlay
      :fringe-bitmap 'flycheck-fringe-bitmap-caret
      :fringe-face 'flycheck-fringe-error
      :error-list-face 'flycheck-error-list-error)
    (flycheck-define-error-level
        'warning
      :severity 100
      :compilation-level 1
      :overlay-category 'flycheck-warning-overlay
      :fringe-bitmap 'flycheck-fringe-bitmap-caret
      :fringe-face 'flycheck-fringe-warning
      :warning-list-face 'flycheck-warning-list-warning)
    (flycheck-define-error-level
        'info
      :severity 100
      :compilation-level 1
      :overlay-category 'flycheck-info-overlay
      :fringe-bitmap 'flycheck-fringe-bitmap-caret
      :fringe-face 'flycheck-fringe-info
      :info-list-face 'flycheck-info-list-info))
```


<a id="orgf230fa3"></a>

### Flymake

```elisp
(with-eval-after-load 'flymake
  (define-fringe-bitmap 'small-right-triangle
    tangonov--fringe-right-triangle)
  (setq flymake-note-bitmap    '(small-right-triangle compilation-info)
        flymake-error-bitmap   '(small-right-triangle compilation-error)
        flymake-warning-bitmap '(small-right-triangle compilation-warning)))
```


<a id="orga2b841c"></a>

### Apply Settings

We need our fringes to be applied after the relevant, related packages load.

```elisp
(when tangonov-enable-custom-fringes
  (fringe-mode '(4 . 0))
  (with-eval-after-load 'bookmark
    (define-fringe-bitmap 'bookmark-fringe-mark
      tangonov--fringe-stub))
  (with-eval-after-load 'flycheck
    (define-fringe-bitmap 'flycheck-fringe-bitmap-caret
      tangonov--fringe-right-triangle)
    (flycheck-define-error-level
          'error
        :severity 100
        :compilation-level 2
        :overlay-category 'flycheck-error-overlay
        :fringe-bitmap 'flycheck-fringe-bitmap-caret
        :fringe-face 'flycheck-fringe-error
        :error-list-face 'flycheck-error-list-error)
      (flycheck-define-error-level
          'warning
        :severity 100
        :compilation-level 1
        :overlay-category 'flycheck-warning-overlay
        :fringe-bitmap 'flycheck-fringe-bitmap-caret
        :fringe-face 'flycheck-fringe-warning
        :warning-list-face 'flycheck-warning-list-warning)
      (flycheck-define-error-level
          'info
        :severity 100
        :compilation-level 1
        :overlay-category 'flycheck-info-overlay
        :fringe-bitmap 'flycheck-fringe-bitmap-caret
        :fringe-face 'flycheck-fringe-info
        :info-list-face 'flycheck-info-list-info))
  (with-eval-after-load 'flymake
    (define-fringe-bitmap 'small-right-triangle
      tangonov--fringe-right-triangle)
    (setq flymake-note-bitmap    '(small-right-triangle compilation-info)
          flymake-error-bitmap   '(small-right-triangle compilation-error)
          flymake-warning-bitmap '(small-right-triangle compilation-warning))))
```


<a id="org5e8f13e"></a>

## Color Definitions

The strategy for writing this theme is to do it as simply as possible. I am only supporting 256 colors (for now).

```elisp
(deftheme tangonov
  "A 256 color dark theme featuring bright pastels.")

(let ((spec '((class color) (min-colors 256)))
      (fg        "#EEFFFF")
      (fg-alt    "#BFC7D5")
      (bg        "#191919")
      (bg-alt    "#232323")
      (red       "#FF7B85")
      (green     "#ABDC88")
      (yellow    "#FFCA41")
      (orange    "#FF996B")
      (blue      "#82AAFF")
      (magenta   "#C792EA")
      (violet    "#BB80B3")
      (cyan      "#89DDFF")
      (teal      "#44b9b1")
      (gray1     "#303030")
      (gray2     "#626262")
      (gray3     "#A8A8A8"))
```


<a id="orgfe6e824"></a>

## Faces

To theme Emacs you must set the faces for every package you would like to see changed. There are so many packages & faces. It is what it is.

```elisp
  (custom-theme-set-faces
   'tangonov
```


<a id="orgc43f92c"></a>

### Ansi Colors

These colors are used by various shell-like packages.

```elisp
   `(ansi-color-black ((,spec (:foregound ,gray1 :background ,gray1))))
   `(ansi-color-blue ((,spec (:foreground ,blue :background ,blue))))
   `(ansi-color-bright-black ((,spec (:foreground ,gray2 :background ,gray2))))
   `(ansi-color-bright-blue ((,spec (:foreground ,blue :background ,blue))))
   `(ansi-color-bright-cyan ((,spec (:foreground ,cyan :background ,cyan))))
   `(ansi-color-bright-green ((,spec (:foreground ,green :background ,green))))
   `(ansi-color-bright-magenta ((,spec (:foreground ,magenta :background ,magenta))))
   `(ansi-color-bright-red ((,spec (:foreground ,red :background ,red))))
   `(ansi-color-bright-white ((,spec (:foreground "#FFFFFF" :background "#FFFFFF"))))
   `(ansi-color-bright-yellow ((,spec (:foreground ,yellow :background ,yellow))))
   `(ansi-color-cyan ((,spec (:foreground ,cyan :background ,cyan))))
   `(ansi-color-green ((,spec (:foreground ,green :background ,green))))
   `(ansi-color-magenta ((,spec (:foreground ,magenta :background ,magenta))))
   `(ansi-color-red ((,spec (:foreground ,red :background ,red))))
   `(ansi-color-white ((,spec (:foreground ,fg :background ,fg))))
   `(ansi-color-yellow ((,spec (:foreground ,yellow :background ,yellow))))
```


<a id="org833c2f5"></a>

### Avy

```elisp
   `(avy-goto-char-timer-face
     ((,spec (:inherit 'isearch))))
   `(avy-background-face ((,spec (:foreground ,(tangonov-darken bg 0.2)))))
   `(avy-lead-face
     ((,spec (:foreground ,red :weight bold))))
   `(avy-lead-face-0
     ((,spec (:inherit 'avy-lead-face :foreground ,yellow))))
   `(avy-lead-face-1
     ((,spec (:inheri avy-lead-face :foreground ,(tangonov-darken yellow 0.4)))))
   `(avy-lead-face-2
     ((,spec (:inherit 'avy-lead-face :foreground
                       ,(tangonov-darken yellow 0.6)))))
```


<a id="org68e597c"></a>

### Company

```elisp
   `(company-echo-common ((,spec (:foreground ,cyan))))
   `(company-tooltip ((,spec (:background ,bg))))
   `(company-tooltip-annotation ((,spec (:foreground ,fg-alt))))
   `(company-tooltip-common ((,spec (:foreground ,cyan))))
   `(company-tooltip-common-selection
     ((,spec (:foreground ,cyan :weight bold))))
   `(company-tooltip-scrollbar-thumb ((,spec (:background ,violet))))
   `(company-tooltip-scrollbar-track ((,spec (:background ,bg-alt))))
   `(company-tooltip-selection ((,spec (:inherit bold :background ,gray1))))
```


<a id="orgc90cceb"></a>

### CSS

```elisp
   `(css-proprietary-property ((,spec (:foreground ,orange))))
   `(css-property ((,spec (:foreground ,green))))
   `(css-selector ((,spec (:foreground ,blue))))
```


<a id="orgb448647"></a>

### Easy Customization

```elisp
   `(custom-button
     ((,spec
       (:foreground ,blue :background ,bg :box '(:line-width 1 :style none)))))
   `(custom-button-unraised
     ((,spec (:foreground ,violet :background
                          ,bg :box '(:line-width 1 :style none)))))
   `(custom-button-pressed-unraised
     ((,spec
       (:foreground ,bg :background
                    ,violet :box '(:line-width 1 :style none)))))
   `(custom-button-pressed
     ((,spec (:foreground ,bg :background
                          ,blue :box '(:line-width 1 :style none)))))
   `(custom-button-mouse
     ((,spec (:foreground ,bg :background ,blue
                          :box '(:line-width 1 :style none)))))
   `(custom-variable-button ((,spec (:foreground ,green :underline t))))
   `(custom-saved
     ((,spec (:foreground ,green :background
                          ,(tangonov-darken green 0.5) :bold bold))))
   `(custom-comment ((,spec (:foreground ,fg))))
   `(custom-comment-tag ((,spec (:foreground ,gray2))))
   `(custom-modified
     ((,spec (:foreground ,blue :background ,(tangonov-darken blue 0.5)))))
   `(custom-variable-tag ((,spec (:foreground ,magenta))))
   `(custom-visibility ((,spec (:foreground ,blue :underline nil))))
   `(custom-group-subtitle ((,spec (:foreground ,red))))
   `(custom-group-tag ((,spec (:foreground ,violet))))
   `(custom-group-tag-1 ((,spec (:foreground ,blue))))
   `(custom-set ((,spec (:foreground ,yellow :background ,bg))))
   `(custom-themed ((,spec (:foreground ,yellow :background ,bg))))
   `(custom-invalid
     ((,spec (:foreground ,red :background ,(tangonov-darken red 0.5)))))
   `(custom-variable-obsolete ((,spec (:foreground ,gray2 :background ,bg))))
   `(custom-state
     ((,spec (:foreground ,green :background ,(tangonov-darken green 0.5)))))
   `(custom-changed ((,spec (:foreground ,blue :background ,bg))))
```


<a id="org01851d6"></a>

### Elfleed

```elisp
   `(elfeed-log-debug-level-face ((,spec (:foreground ,gray2))))
   `(elfeed-log-error-level-face ((,spec (:inherit 'error))))
   `(elfeed-log-info-level-face ((,spec (:inherit 'success))))
   `(elfeed-log-warn-level-face ((,spec (:inherit 'warning))))
   `(elfeed-search-date-face ((,spec (:foreground ,violet))))
   `(elfeed-search-feed-face ((,spec (:foreground ,blue))))
   `(elfeed-search-tag-face ((,spec (:foreground ,gray2))))
   `(elfeed-search-title-face ((,spec (:foreground ,gray2))))
   `(elfeed-search-filter-face ((,spec (:foreground ,violet))))
   `(elfeed-search-unread-count-face ((,spec (:foreground ,yellow))))
   `(elfeed-search-unread-title-face ((,spec (:foreground ,fg :weight bold))))
```


<a id="org8e9c7f0"></a>

### EWW

```elisp
   `(eww-form-checkbox ((,spec (:inherit 'eww-form-file))))
   `(eww-form-file   ((,spec (:inherit 'eww-form-submit :background ,bg-alt))))
   `(eww-form-select ((,spec (:inherit 'eww-form-submit :background ,bg-alt))))
   `(eww-form-submit
     ((,spec (:inherit 'eww-form-text :box
                       `(:line-width 2 :style released-button)
                       :background ,gray1))))
   `(eww-form-text
     ((,spec (:box `(:line-width 1 :color ,gray2)
                   ,bg :foreground ,fg :distant-foreground ,bg))))
   `(eww-form-textarea ((,spec (:inherit 'eww-form-text))))
   `(eww-invalid-certificate ((,spec (:foreground ,red))))
   `(eww-valid-certificate ((,spec (:foreground ,cyan))))
```


<a id="orge443435"></a>

### Emacs

Set the basic faces for the editor. Many of these faces are used commonly throughout Emacs. Some of them derive other faces.

```elisp
   `(default ((,spec (:background ,bg :foreground ,fg))))
   `(bold ((,spec (:weight bold))))
   `(italic ((,spec (:slant italic))))
   `(bold-italic ((,spec (:weight bold :slant italic))))
   `(underline ((,spec (:underline t))))
   `(shadow ((,spec (:foreground ,gray2))))
   `(link ((,spec (:foreground ,blue :weight bold :underline t))))
   `(link-visited ((,spec (:inherit 'link :foreground ,magenta))))
   `(highlight ((,spec (:background ,gray1 :weight bold))))
   `(match ((,spec (:foreground
                    ,green :background ,(tangonov-darken green 0.5)))))
   `(region ((,spec (:foreground
                     ,cyan :background ,(tangonov-darken cyan 0.5)))))
   `(secondary-selection ((,spec (:background ,gray2 :foreground ,fg))))
   `(lazy-highlight ((,spec
                      (:foreground ,blue :background
                                   ,(tangonov-darken blue 0.5)))))
   `(error ((,spec (:foreground ,red))))
   `(warning ((,spec (:foreground ,yellow))))
   `(success ((,spec (:foreground ,green))))
   `(escape-glyph ((,spec (:foreground ,orange))))
   `(homoglyph ((,spec (:foreground ,orange))))
   `(vertical-border ((,spec (:foreground ,gray1))))
   `(cursor ((,spec (:background ,yellow))))
   `(minibuffer-prompt ((,spec (:foreground ,yellow))))
   `(line-number-current-line ((,spec (:foreground ,cyan :background ,gray1))))
   `(completions-common-part ((,spec (:foreground ,cyan))))
   `(completions-first-difference ((,spec (:foreground ,yellow))))
   `(trailing-whitespace ((,spec (:background ,red))))
   `(whitespace-trailing ((,spec (:background ,red))))
   `(bookmark-face ((,spec (:foreground ,orange))))
   `(tool-bar ((,spec (:foreground ,fg :background ,bg-alt))))
   `(tooltip ((,spec (:foreground ,fg :background ,bg-alt))))
```


<a id="org8fee7a6"></a>

### Email

There are many packages that cobble together different Email & RSS interfaces. Many of them look to `message-mode` or `gnus` for faces. Others have their own opinions.

1.  Message Mode

    ```elisp
       `(message-header-name ((,spec (:foreground ,green))))
       `(message-header-subject ((,spec (:foreground ,cyan :weight bold))))
       `(message-header-to ((,spec (:foreground ,cyan :weight bold))))
       `(message-header-cc
         ((,spec (:inherit 'message-header-to
                           :foreground ,(tangonov-darken cyan 0.15)))))
       `(message-header-other ((,spec (:foreground ,violet))))
       `(message-header-newsgroups ((,spec (:foreground ,yellow))))
       `(message-header-xheader ((,spec (:foreground ,gray3))))
       `(message-separator ((,spec (:foreground ,gray2))))
       `(message-mml ((,spec (:foreground ,gray2 :slant italic))))
    ```

2.  GNUs

    ```elisp
       `(gnus-group-mail-1 ((,spec (:weight bold :foreground ,fg))))
       `(gnus-group-mail-2 ((,spec (:inherit 'gnus-group-mail-1))))
       `(gnus-group-mail-3 ((,spec (:inherit 'gnus-group-mail-1))))
       `(gnus-group-mail-1-empty ((,spec (:foreground ,gray3))))
       `(gnus-group-mail-2-empty ((,spec (:inherit 'gnus-group-mail-1-empty))))
       `(gnus-group-mail-3-empty ((,spec (:inherit 'gnus-group-mail-1-empty))))
       `(gnus-group-news-1 ((,spec (:inherit 'gnus-group-mail-1))))
       `(gnus-group-news-2 ((,spec (:inherit 'gnus-group-news-1))))
       `(gnus-group-news-3 ((,spec (:inherit 'gnus-group-news-1))))
       `(gnus-group-news-4 ((,spec (:inherit 'gnus-group-news-1))))
       `(gnus-group-news-5 ((,spec (:inherit 'gnus-group-news-1))))
       `(gnus-group-news-6 ((,spec (:inherit 'gnus-group-news-1))))
       `(gnus-group-news-1-empty ((,spec (:inherit 'gnus-group-mail-1-empty))))
       `(gnus-group-news-2-empty ((,spec (:inherit 'gnus-group-news-1-empty))))
       `(gnus-group-news-3-empty ((,spec (:inherit 'gnus-group-news-1-empty))))
       `(gnus-group-news-4-empty ((,spec (:inherit 'gnus-group-news-1-empty))))
       `(gnus-group-news-5-empty ((,spec (:inherit 'gnus-group-news-1-empty))))
       `(gnus-group-news-6-empty ((,spec (:inherit 'gnus-group-news-1-empty))))
       `(gnus-group-mail-low ((,spec (:inherit 'gnus-group-mail-1 :weight normal))))
       `(gnus-group-mail-low-empty ((,spec (:inherit 'gnus-group-mail-1-empty))))
       `(gnus-group-news-low
         ((,spec (:inherit 'gnus-group-mail-1 :foreground ,gray3))))
       `(gnus-group-news-low-empty
         ((,spec (:inherit 'gnus-group-news-low :weight normal))))
       `(gnus-header-content ((,spec (:inherit 'message-header-other))))
       `(gnus-header-from ((,spec (:inherit 'message-header-other))))
       `(gnus-header-name ((,spec (:inherit 'message-header-name))))
       `(gnus-header-newsgroups ((,spec (:inherit 'message-header-other))))
       `(gnus-header-subject ((,spec (:inherit 'message-header-subject))))
       `(gnus-summary-cancelled ((,spec (:foreground ,red :strike-through t))))
       `(gnus-summary-high-ancient
         ((,spec (:foreground ,(tangonov-lighten gray3 0.2) :inherit 'italic))))
       `(gnus-summary-high-read
         ((,spec (:foreground ,(tangonov-lighten fg 0.2)))))
       `(gnus-summary-high-ticked
         ((,spec (:foreground ,(tangonov-lighten magenta 0.2)))))
       `(gnus-summary-high-unread
         ((,spec (:foreground ,(tangonov-lighten green 0.2)))))
       `(gnus-summary-low-ancient
         ((,spec (:foreground ,(tangonov-darken gray3 0.2) :inherit 'italic))))
       `(gnus-summary-low-read ((,spec (:foreground ,(tangonov-darken fg 0.2)))))
       `(gnus-summary-low-ticked
         ((,spec (:foreground ,(tangonov-darken magenta 0.2)))))
       `(gnus-summary-low-unread
         ((,spec (:foreground ,(tangonov-darken green 0.2)))))
       `(gnus-summary-normal-ancient
         ((,spec (:foreground ,gray3 :inherit 'italic))))
       `(gnus-summary-normal-read ((,spec (:foreground ,fg))))
       `(gnus-summary-normal-ticked ((,spec (:foreground ,magenta))))
       `(gnus-summary-normal-unread ((,spec (:foreground ,green :inherit 'bold))))
       `(gnus-summary-selected ((,spec (:foreground ,blue :weight bold))))
       `(gnus-cite-1 ((,spec (:foreground ,violet))))
       `(gnus-cite-2 ((,spec (:foreground ,yellow))))
       `(gnus-cite-3 ((,spec (:foreground ,magenta))))
       `(gnus-cite-4 ((,spec (:foreground ,green))))
       `(gnus-cite-5 ((,spec (:foreground ,green))))
       `(gnus-cite-6 ((,spec (:foreground ,green))))
       `(gnus-cite-7 ((,spec (:foreground ,magenta))))
       `(gnus-cite-8 ((,spec (:foreground ,magenta))))
       `(gnus-cite-9 ((,spec (:foreground ,magenta))))
       `(gnus-cite-10 ((,spec (:foreground ,yellow))))
       `(gnus-cite-11 ((,spec (:foreground ,yellow))))
       `(gnus-signature ((,spec (:foreground ,yellow))))
       `(gnus-x-face ((,spec (:background ,gray3 :foreground ,fg))))
    ```

3.  Notmuch

    ```elisp
       `(notmuch-crypto-decryption ((,spec (:foreground ,magenta))))
       `(notmuch-crypto-signature-bad ((,spec (:foreground ,red))))
       `(notmuch-crypto-signature-good ((,spec (:foreground ,green))))
       `(notmuch-crypto-signature-good-key ((,spec (:foreground ,orange))))
       `(notmuch-crypto-signature-unknown ((,spec (:foreground ,red))))
       `(notmuch-message-summary-face
         ((,spec (:background ,bg-alt :overline ,gray2))))
       `(notmuch-search-count ((,spec (:foreground ,gray2))))
       `(notmuch-search-date ((,spec (:foreground ,orange))))
       `(notmuch-search-flagged-face
         ((,spec (:foreground ,(tangonov-darken red 0.5)))))
       `(notmuch-search-matching-authors ((,spec (:foreground ,blue))))
       `(notmuch-search-non-matching-authors ((,spec (:foreground ,fg))))
       `(notmuch-search-subject ((,spec (:foreground ,fg))))
       `(notmuch-search-unread-face ((,spec (:weight bold))))
       `(notmuch-tag-added ((,spec (:foreground ,green :weight normal))))
       `(notmuch-tag-deleted ((,spec (:foreground ,red :weight normal))))
       `(notmuch-tag-face ((,spec (:foreground ,yellow :weight normal))))
       `(notmuch-tag-flagged ((,spec (:foreground ,yellow :weight normal))))
       `(notmuch-tag-unread ((,spec (:foreground ,yellow :weight normal))))
       `(notmuch-tree-match-author-face ((,spec (:foreground ,blue :weight bold))))
       `(notmuch-tree-match-date-face ((,spec (:foreground ,orange :weight bold))))
       `(notmuch-tree-match-face ((,spec (:foreground ,fg))))
       `(notmuch-tree-match-subject-face ((,spec (:foreground ,fg))))
       `(notmuch-tree-match-tag-face ((,spec (:foreground ,yellow))))
       `(notmuch-tree-match-tree-face ((,spec (:foreground ,gray2))))
       `(notmuch-tree-no-match-author-face ((,spec (:foreground ,blue))))
       `(notmuch-tree-no-match-date-face ((,spec (:foreground ,orange))))
       `(notmuch-tree-no-match-face ((,spec (:foreground ,gray3))))
       `(notmuch-tree-no-match-subject-face ((,spec (:foreground ,gray3))))
       `(notmuch-tree-no-match-tag-face ((,spec (:foreground ,yellow))))
       `(notmuch-tree-no-match-tree-face ((,spec (:foreground ,yellow))))
       `(notmuch-wash-cited-text ((,spec (:foreground ,gray1))))
       `(notmuch-wash-toggle-button ((,spec (:foreground ,fg))))
    ```


<a id="orgea1a24a"></a>

### ERC

```elisp
   `(erc-button ((,spec (:weight bold :underline t))))
   `(erc-default-face ((,spec (:inherit 'default))))
   `(erc-action-face ((,spec (:weight bold))))
   `(erc-command-indicator-face ((,spec (:weight bold))))
   `(erc-direct-msg-face ((,spec (:foreground ,magenta))))
   `(erc-error-face ((,spec (:inherit 'error))))
   `(erc-header-line
     ((,spec (:background ,(tangonov-darken bg-alt 0.15) :foreground ,cyan))))
   `(erc-input-face ((,spec (:foreground ,green))))
   `(erc-current-nick-face ((,spec (:foreground ,green :weight bold))))
   `(erc-timestamp-face ((,spec (:foreground ,blue :weight bold))))
   `(erc-nick-default-face ((,spec (:weight bold))))
   `(erc-nick-msg-face ((,spec (:foreground ,magenta))))
   `(erc-nick-prefix-face ((,spec (:inherit 'erc-nick-default-face))))
   `(erc-my-nick-face ((,spec (:foreground ,green :weight bold))))
   `(erc-my-nick-prefix-face ((,spec (:inherit 'erc-my-nick-face))))
   `(erc-notice-face ((,spec (:foreground ,gray2))))
   `(erc-prompt-face ((,spec (:foreground ,cyan :weight bold))))
```


<a id="org17ffb0f"></a>

### Evil

Support for various Evil related features/packages.

```elisp
   `(evil-ex-info ((,spec (:foreground ,red :slant italic))))
   `(evil-ex-search
     ((,spec (:background ,gray1 :foreground ,cyan :weight bold))))
   `(evil-ex-substitute-matches
     ((,spec (:background ,gray1 :foreground
                          ,red :weight bold :strike-through t))))
   `(evil-ex-substitute-replacement
     ((,spec (:background ,gray1 :foreground ,green :weight bold))))
   `(evil-search-highlight-persist-highlight-face
     ((,spec (:inherit 'lazy-highlight))))
```

1.  evil-mc

    ```elisp
       `(evil-mc-cursor-default-face
         ((,spec (:background ,magenta :foreground ,gray1 :inverse-video nil))))
       `(evil-mc-region-face ((,spec (:inherit 'region))))
       `(evil-mc-cursor-bar-face
         ((,spec (:height 1 :background ,magenta :foreground ,gray1))))
       `(evil-mc-cursor-hbar-face ((,spec (:underline `(:color ,cyan)))))
    ```

2.  evil-snipe

    ```elisp
       `(evil-snipe-first-match-face
         ((,spec (:foreground ,blue :background
                              ,(tangonov-darken blue 0.5) :weight bold))))
       `(evil-snipe-matches-face
         ((,spec (:foreground highlight :underline t :weight bold))))
    ```

3.  evil-goggles

    ```elisp
       `(evil-goggles-delete-face
         ((,spec (:foreground ,(tangonov-darken red 0.5) :background ,red))))
       `(evil-goggles-paste-face
         ((,spec (:foreground ,(tangonov-darken green 0.5) :background ,green))))
       `(evil-goggles-undo-redo-add-face ((,spec (:inherit 'evil-goggles-paste-face))))
       `(evil-goggles-undo-redo-remove-face ((,spec (:inherit 'evil-goggles-delete-face))))
       `(evil-goggles-record-macro-face
         ((,spec (:foreground ,(tangonov-darken yellow 0.5) :background ,yellow))))
    ```


<a id="org445868f"></a>

### Font Lock Faces

These faces end up being inherited by *many* major modes for highlighting.

```elisp
   ;; Font Lock
   `(font-lock-warning-face ((,spec (:inherit 'warning))))
   `(font-lock-function-name-face ((,spec (:foreground ,blue))))
   `(font-lock-variable-name-face ((,spec (:foreground ,yellow))))
   `(font-lock-keyword-face ((,spec (:foreground ,cyan))))
   `(font-lock-comment-face ((,spec (:foreground ,gray2))))
   `(font-lock-type-face ((,spec (:foreground ,magenta))))
   `(font-lock-constant-face ((,spec (:foreground ,orange))))
   `(font-lock-builtin-face ((,spec (:foreground ,cyan))))
   `(font-lock-string-face ((,spec (:foreground ,green))))
   `(font-lock-doc-face ((,spec (:foreground ,gray2))))
   `(font-lock-negation-char-face ((,spec (:foreground ,orange))))
```


<a id="orgdbf3613"></a>

### Goggles

```elisp
   `(goggles-changed ((,spec (:background ,cyan))))
   `(goggles-added ((,spec (:background ,green))))
   `(goggles-removed ((,spec (:background ,red))))
```


<a id="orgd01a68e"></a>

### Hydra

```elisp
   `(hydra-face-red ((,spec (:foreground ,red :weight bold))))
   `(hydra-face-blue ((,spec (:foreground ,blue :weight bold))))
   `(hydra-face-amaranth ((,spec (:foreground ,magenta :weight bold))))
   `(hydra-face-pink ((,spec (:foreground ,violet :weight bold))))
   `(hydra-face-teal ((,spec (:foreground ,teal :weight bold))))
```


<a id="orgc609bbb"></a>

### Inf-Ruby

```elisp
   `(inf-ruby-result-overlay-face
     ((,spec (:foreground ,cyan :background
                          ,bg-alt :box (:line-width 1 :color ,cyan)))))
```


<a id="org4823504"></a>

### ISearch

```elisp
   `(isearch ((,spec (:inherit 'match :weight bold))))
   `(isearch-fail ((,spec (:background ,red :foreground ,gray1 :weight bold))))
```


<a id="orgd507d67"></a>

### Keycast

```elisp
   `(keycast-key
     ((,spec (:weight bold
                      :background ,green
                      :foreground ,bg))))
   `(keycast-command ((,spec (:foreground ,green :weight bold))))
```


<a id="orgac01855"></a>

### Linters

1.  Flymake

    ```elisp
       `(flymake-error ((,spec (:underline (:style wave :color ,red)))))
       `(flymake-note ((,spec (:underline (:style wave :color ,green)))))
       `(flymake-warning ((,spec (:underline (:style wave :color ,orange)))))
    ```

2.  Flycheck

    ```elisp
       `(flycheck-error ((,spec (:underline (:style wave :color ,red)))))
       `(flycheck-warning ((,spec (:underline (:style wave :color ,yellow)))))
       `(flycheck-info ((,spec (:underline (:style wave :color ,green)))))
       `(flycheck-fringe-error ((,spec (:inherit 'fringe :foreground ,red))))
       `(flycheck-fringe-warning ((,spec (:inherit 'fringe :foreground ,yellow))))
       `(flycheck-fringe-info ((,spec (:inherit 'fringe :foreground ,green))))
       `(flycheck-posframe-face ((,spec (:inherit 'default))))
       `(flycheck-posframe-background-face ((,spec (:background ,bg-alt))))
       `(flycheck-posframe-error-face
         ((,spec (:inherit 'flycheck-posframe-face :foreground ,red))))
       `(flycheck-posframe-info-face
         ((,spec (:inherit 'flycheck-posframe-face :foreground ,fg))))
       `(flycheck-posframe-warning-face
         ((,spec (:inherit 'flycheck-posframe-face :foreground ,yellow))))
    ```

3.  Flyspell

    ```elisp
       `(flyspell-incorrect
         ((,spec (:underline (:style wave :color ,red) :inherit 'unspecified))))
       `(flyspell-duplicate
         ((,spec (:underline (:style wave :color ,yellow) :inherit 'unspecified))))
    ```


<a id="org741a765"></a>

### LSP

1.  Eglot

    ```elisp
       `(eglot-highlight-symbol-face ((,spec (:weight bold :background ,gray1))))
    ```

2.  Eldoc Box

    ```elisp
       `(eldoc-box-border ((,spec (:background ,fg-alt))))
    ```


<a id="org7523082"></a>

### Mode-line

Set faces for the top and bottom "bars."

```elisp
   `(mode-line
     ((,spec (:foreground ,fg-alt :background ,bg-alt :box
                          (:line-width (2 . 2) :color ,bg-alt)))))
   `(mode-line-inactive
     ((,spec (:inherit 'mode-line :foreground ,gray2 :background ,bg))))
   `(mode-line-highlight ((,spec (:box (:line-width (2 . 2) :color ,magenta)))))
   `(mode-line-buffer-id ((,spec (:foreground ,fg :weight bold))))
```


<a id="orga01dc3d"></a>

### Org Mode

Org-mode has many faces. It takes some work to make them consistent in buffers and in the agenda.

1.  Documents

    ```elisp
       `(org-block ((,spec (:background ,bg-alt))))
       `(org-block-background ((,spec (:background ,bg-alt))))
       `(org-block-begin-line ((,spec (:foreground ,gray2 :background ,bg))))
       `(org-level-1 ((,spec (:inherit bold :foreground ,green))))
       `(org-level-2 ((,spec (:inherit bold :foreground ,yellow))))
       `(org-level-3 ((,spec (:inherit bold :foreground ,red))))
       `(org-level-4 ((,spec (:inherit bold :foreground ,cyan))))
       `(org-level-5 ((,spec (:inherit bold :foreground ,blue))))
       `(org-level-6 ((,spec (:inherit bold :foreground ,magenta))))
       `(org-level-7 ((,spec (:inherit bold :foreground ,teal))))
       `(org-level-8 ((,spec (:inherit bold :foreground ,violet))))
       `(org-headline-done ((,spec (:foreground ,gray2))))
       `(org-table ((,spec (:foreground ,magenta))))
       `(org-todo ((,spec (:foreground ,orange))))
       `(org-done ((,spec (:foreground ,gray2))))
       `(org-drawer ((,spec (:foreground ,gray2))))
       `(org-meta-line ((,spec (:foreground ,gray2))))
       `(org-special-keyword ((,spec (:foreground ,gray3))))
       `(org-property-value ((,spec (:foreground ,red))))
       `(org-tag ((,spec (:foreground ,fg-alt))))
       `(org-verbatim ((,spec (:foreground ,green))))
       `(org-code ((,spec (:foreground ,orange :background ,bg-alt))))
       `(org-document-info-keyword ((,spec (:foreground ,red))))
       `(org-document-info ((,spec (:foreground ,fg-alt))))
       `(org-document-title ((,spec (:foreground ,yellow))))
       `(org-date ((,spec (:foreground ,yellow))))
       `(org-checkbox ((,spec (:foreground ,orange))))
       `(org-checkbox-statistics-todo ((,spec (:inherit 'org-checkbox))))
       `(org-checkbox-statistics-done ((,spec (:inherit 'org-done))))
    ```

2.  Agenda

    ```elisp
       `(org-agenda-done ((,spec (:inherit 'org-done))))
       `(org-agenda-clocking
         ((,spec (:background ,(tangonov-darken cyan 0.5) :extend t))))
       `(org-time-grid ((,spec (:foreground ,gray2))))
       `(org-imminent-deadline ((,spec (:foreground ,yellow))))
       `(org-upcoming-deadline ((,spec (:foreground ,teal))))
       `(org-agenda-dimmed-todo-face ((,spec (:foreground ,gray3))))
    ```

3.  Habit

    Faces for tracking habits with the agenda view.
    
    ```elisp
       `(org-habit-clear-face ((,spec (:weight bold :background ,gray2))))
       `(org-habit-clear-future-face ((,spec (:weight bold :background ,gray3))))
       `(org-habit-ready-face
         ((,spec (:weight bold :background ,(tangonov-darken blue 0.5)))))
       `(org-habit-ready-future-face
         ((,spec (:weight bold :background ,(tangonov-darken blue 0.3)))))
       `(org-habit-alert-face
         ((,spec (:weight bold :background ,(tangonov-darken yellow 0.5)))))
       `(org-habit-alert-future-face
         ((,spec (:weight bold :background ,(tangonov-darken yellow 0.3)))))
       `(org-habit-overdue-face
         ((,spec (:weight bold :background ,(tangonov-darken red 0.5)))))
       `(org-habit-overdue-future-face
         ((,spec (:weight bold :background ,(tangonov-darken red 0.3)))))
    ```

4.  Journal

    ```elisp
       `(org-journal-highlight ((,spec (:foreground ,violet))))
       `(org-journal-calendar-entry-face
         ((,spec (:foreground ,magenta :slant italic))))
       `(org-journal-calendar-scheduled-face
         ((,spec (:foreground ,red :slant italic))))
    ```

5.  Podomoro

    ```elisp
       `(org-pomodoro-mode-line ((,spec (:foreground ,fg))))
       `(org-pomodoro-mode-line-overtime
         ((,spec (:foreground ,yellow :weight bold))))
    ```

6.  Mode-Line

    ```elisp
       `(org-mode-line-clock ((,spec (:foreground ,fg))))
       `(org-mode-line-clock-overrun ((,spec (:inherit error))))
    ```

7.  Ref faces

    ```elisp
       `(org-ref-acronym-face ((,spec (:foreground ,violet))))
       `(org-ref-cite-face
         ((,spec (:foreground ,yellow :weight light :underline t))))
       `(org-ref-glossary-face ((,spec (:foreground ,magenta))))
       `(org-ref-label-face ((,spec (:foreground ,blue))))
       `(org-ref-ref-face ((,spec (:inherit 'link :foreground ,teal))))
    ```


<a id="org2312c4a"></a>

### Rainbow Delimiters

```elisp
   `(rainbow-delimiters-depth-1-face ((,spec (:foreground ,magenta))))
   `(rainbow-delimiters-depth-2-face ((,spec (:foreground ,orange))))
   `(rainbow-delimiters-depth-3-face ((,spec (:foreground ,green))))
   `(rainbow-delimiters-depth-4-face ((,spec (:foreground ,cyan))))
   `(rainbow-delimiters-depth-5-face ((,spec (:foreground ,violet))))
   `(rainbow-delimiters-depth-6-face ((,spec (:foreground ,yellow))))
   `(rainbow-delimiters-depth-7-face ((,spec (:foreground ,blue))))
   `(rainbow-delimiters-depth-8-face ((,spec (:foreground ,teal))))
   `(rainbow-delimiters-depth-9-face ((,spec (:foreground ,red))))
```


<a id="org2bb0704"></a>

### RJSX Mode

```elisp
   `(rjsx-tag ((,spec (:foreground ,red))))
   `(rjsx-attr ((,spec (:foreground ,yellow :slant italic :weight medium))))
   `(rjsx-tag-bracket-face ((,spec (:foreground ,cyan))))
```


<a id="orgdb6353e"></a>

### Shells

1.  Eshell

    ```elisp
       `(eshell-prompt ((,spec (:foreground ,magenta :weight bold))))
       `(eshell-ls-archive ((,spec (:foreground ,gray2))))
       `(eshell-ls-backup ((,spec (:foreground ,yellow))))
       `(eshell-ls-clutter ((,spec (:foreground ,red))))
       `(eshell-ls-directory ((,spec (:foreground ,blue))))
       `(eshell-ls-executable ((,spec (:foreground ,green))))
       `(eshell-ls-missing ((,spec (:foreground ,red))))
       `(eshell-ls-product ((,spec (:foreground ,orange))))
       `(eshell-ls-readonly ((,spec (:foreground ,orange))))
       `(eshell-ls-special ((,spec (:foreground ,violet))))
       `(eshell-ls-symlink ((,spec (:foreground ,cyan))))
       `(eshell-ls-unreadable ((,spec (:foreground ,gray3))))
    ```

2.  Vterm

    ```elisp
       `(vterm-color-black
         ((,spec (:background ,gray1 :foreground ,(tangonov-lighten gray1 0.2)))))
       `(vterm-color-red
         ((,spec (:background ,red :foreground ,(tangonov-lighten red 0.2)))))
       `(vterm-color-green
         ((,spec (:background ,green :foreground ,(tangonov-lighten green 0.2)))))
       `(vterm-color-yellow
         ((,spec (:background ,yellow :foreground ,(tangonov-lighten yellow 0.2)))))
       `(vterm-color-blue
         ((,spec (:background ,blue :foreground ,(tangonov-lighten blue 0.2)))))
       `(vterm-color-magenta
         ((,spec (:background ,magenta :foreground
                              ,(tangonov-lighten violet 0.2)))))
       `(vterm-color-cyan
         ((,spec (:background ,cyan :foreground ,(tangonov-lighten cyan 0.2)))))
       `(vterm-color-white ((,spec (:background ,fg :foreground ,gray3))))
    ```


<a id="org357c097"></a>

### Tab Bar/Line

```elisp
   `(tab-line ((,spec (:background ,bg-alt))))
   `(tab-line-tab ((,spec (:background ,bg-alt :foreground ,fg-alt))))
   `(tab-line-tab-inactive
     ((,spec (:inherit 'tab-line-tab
                       :background ,bg-alt :foreground ,gray2))))
   `(tab-line-tab-inactive-alternate
     ((,spec (:inherit 'tab-line-tab-inactive))))
   `(tab-line-tab-current ((,spec (:background ,bg-alt :foreground ,fg))))
   `(tab-line-highlight ((,spec (:inherit 'tab-line-tab))))
   `(tab-line-close-highlight ((,spec (:foreground ,cyan))))
   `(tab-bar ((,spec (:inherit tab-line))))
   `(tab-bar-tab ((,spec (:inherit tab-line-tab))))
   `(tab-bar-tab-inactive ((,spec (:inherit tab-line-tab-inactive))))
```


<a id="org1823062"></a>

### Typescript.el

```elisp
   `(typescript-jsdoc-tag ((,spec (:foreground ,magenta))))
   `(typescript-jsdoc-type ((,spec (:foreground ,gray3))))
   `(typescript-jsdoc-value ((,spec (:foreground ,cyan))))
```


<a id="org759dfcc"></a>

### Version Control

Set the faces for several version-control related packages.

1.  Diff Mode

    ```elisp
       `(diff-added ((,spec
                      (:foreground ,green :background
                                   ,(tangonov-darken green 0.5)))))
       `(diff-changed
         ((,spec (:foreground ,blue :background ,(tangonov-darken blue 0.5)))))
       `(diff-context ((,spec (:foreground ,gray3))))
       `(diff-removed
         ((,spec (:foreground ,red :background ,(tangonov-darken red 0.5)))))
       `(diff-header ((,spec (:foreground ,cyan))))
       `(diff-file-header ((,spec (:foreground ,blue :background ,bg))))
       `(diff-hunk-header ((,spec (:foreground ,violet))))
       `(diff-refine-added ((,spec (:inherit 'diff-added :inverse-video t))))
       `(diff-refine-changed ((,spec (:inherit 'diff-changed :inverse-video t))))
       `(diff-refine-removed ((,spec (:inherit 'diff-removed :inverse-video t))))
    ```

2.  Diff-hl

    ```elisp
       `(diff-hl-change ((,spec (:background ,blue :foreground ,blue))))
       `(diff-hl-delete ((,spec (:background ,red :foreground ,red))))
       `(diff-hl-insert ((,spec (:background ,green :foreground ,green))))
    ```

3.  Ediff

    ```elisp
       `(ediff-fine-diff-A ((,spec
                             (:background
                              ,(tangonov-blend cyan bg 0.7) :weight bold :extend))))
       `(ediff-fine-diff-B ((,spec (:inherit 'ediff-fine-diff-A))))
       `(ediff-fine-diff-C ((,spec (:inherit 'ediff-fine-diff-A))))
       `(ediff-current-diff-A
         ((,spec (:background ,(tangonov-blend cyan bg 0.3) :extend t))))
       `(ediff-current-diff-B ((,spec (:inherit 'ediff-current-diff-A))))
       `(ediff-current-diff-C ((,spec (:inherit 'ediff-current-diff-A))))
       `(ediff-even-diff-A ((,spec (:inherit 'hl-line))))
       `(ediff-even-diff-B ((,spec (:inherit 'ediff))))
       `(ediff-even-diff-C ((,spec (:inherit 'ediff-even-diff-A))))
       `(ediff-odd-diff-A ((,spec (:inherit 'ediff-even-diff-A))))
       `(ediff-odd-diff-B ((,spec (:inherit 'ediff-odd-diff-A))))
       `(ediff-odd-diff-C ((,spec (:inherit 'ediff-odd-diff-A))))
    ```

4.  Magit

    Magit is a monster sized package with many, many faces
    
    ```elisp
       `(magit-bisect-bad ((,spec (:foreground ,red))))
       `(magit-bisect-good ((,spec (:foreground ,green))))
       `(magit-bisect-skip ((,spec (:foreground ,orange))))
       `(magit-blame-hash ((,spec (:foreground ,cyan))))
       `(magit-blame-date ((,spec (:foreground ,red))))
       `(magit-blame-heading
         ((,spec (:foreground ,orange :background ,gray3 :extend t))))
       `(magit-branch-current ((,spec (:foreground ,blue))))
       `(magit-branch-local ((,spec (:foreground ,cyan))))
       `(magit-branch-remote ((,spec (:foreground ,green))))
       `(magit-cherry-equivalent ((,spec (:foreground ,violet))))
       `(magit-cherry-unmatched ((,spec (:foreground ,cyan))))
       `(magit-diff-added
         ((,spec (:foreground ,(tangonov-darken green 0.2) :background
                              ,(tangonov-blend green bg 0.1) :extend t))))
       `(magit-diff-added-highlight
         ((,spec (:foreground ,green :background
                              ,(tangonov-blend green bg 0.2)
                              :weight bold :extend t))))
       `(magit-diff-base
         ((,spec (:foreground ,(tangonov-darken orange 0.2) :background
                              ,(tangonov-blend orange bg 0.1) :extend t))))
       `(magit-diff-base-highlight
         ((,spec (:foreground ,orange :background
                              ,(tangonov-blend orange bg 0.2) :weight
                              bold :extend t))))
       `(magit-diff-context
         ((,spec (:foreground ,(tangonov-darken fg 0.4) :background
                              ,bg :extend t))))
       `(magit-diff-context-highlight
         ((,spec (:foreground ,fg :background ,bg-alt :extend t))))
       `(magit-diff-file-heading
         ((,spec (:foreground ,fg :weight bold :extend t))))
       `(magit-diff-file-heading-selection
         ((,spec (:foreground ,magenta :background
                              ,(tangonov-darken blue 0.5) :weight bold :extend t))))
       `(magit-diff-hunk-heading
         ((,spec (:foreground ,bg :background
                              ,(tangonov-blend violet bg 0.3) :extend t))))
       `(magit-diff-hunk-heading-highlight
         ((,spec (:foreground ,bg :background ,violet :weight bold :extend t))))
       `(magit-diff-lines-heading
         ((,spec (:foreground ,yellow :background ,red :extend t :extend t))))
       `(magit-diff-removed
         ((,spec (:foreground ,(tangonov-darken red 0.2) :background
                              ,(tangonov-blend red bg 0.1) :extend t))))
       `(magit-diff-removed-highlight
         ((,spec (:foreground ,red :background
                              ,(tangonov-blend red bg 0.2)
                              :weight bold :extend t))))
       `(magit-diffstat-added ((,spec (:foreground ,green))))
       `(magit-diffstat-removed ((,spec (:foreground ,red))))
       `(magit-dimmed ((,spec (:foreground ,gray2))))
       `(magit-hash ((,spec (:foreground ,gray2))))
       `(magit-header-line
         ((,spec (:background ,bg-alt :foreground ,yellow :weight bold))))
       `(magit-filename ((,spec (:foreground ,violet))))
       `(magit-log-author ((,spec (:foreground ,orange))))
       `(magit-log-date ((,spec (:foreground ,blue))))
       `(magit-log-graph ((,spec (:foreground ,gray2))))
       `(magit-process-ng ((,spec (:inherit 'error))))
       `(magit-process-ok ((,spec (:inherit 'success))))
       `(magit-reflog-amend ((,spec (:foreground ,magenta))))
       `(magit-reflog-checkout ((,spec (:foreground ,blue))))
       `(magit-reflog-cherry-pick ((,spec (:foreground ,green))))
       `(magit-reflog-commit ((,spec (:foreground ,green))))
       `(magit-reflog-merge ((,spec (:foreground ,green))))
       `(magit-reflog-other ((,spec (:foreground ,cyan))))
       `(magit-reflog-rebase ((,spec (:foreground ,magenta))))
       `(magit-reflog-remote ((,spec (:foreground ,cyan))))
       `(magit-reflog-reset ((,spec (:inherit 'error))))
       `(magit-refname ((,spec (:foreground ,gray2))))
       `(magit-section-heading
         ((,spec (:foreground ,blue :weight bold :extend t))))
       `(magit-section-heading-selection
         ((,spec (:foreground ,orange :weight bold :extend t))))
       `(magit-section-highlight ((,spec (:inherit 'hl-line))))
       `(magit-section-secondary-heading
         ((,spec (:foreground ,violet :weight bold :extend t))))
       `(magit-sequence-drop ((,spec (:foreground ,red))))
       `(magit-sequence-head ((,spec (:foreground ,blue))))
       `(magit-sequence-part ((,spec (:foreground ,orange))))
       `(magit-sequence-stop ((,spec (:foreground ,green))))
       `(magit-signature-bad ((,spec (:inherit 'error))))
       `(magit-signature-error ((,spec (:inherit 'error))))
       `(magit-signature-expired ((,spec (:foreground ,orange))))
       `(magit-signature-good ((,spec (:inherit 'success))))
       `(magit-signature-revoked ((,spec (:foreground ,magenta))))
       `(magit-signature-untrusted ((,spec (:foreground ,yellow))))
       `(magit-tag ((,spec (:foreground ,yellow))))
    ```


<a id="org67b531a"></a>

### Web Mode

```elisp
   `(web-mode-html-tag-face ((,spec (:foreground ,red))))
   `(web-mode-html-attr-equal-face ((,spec (:foreground ,cyan))))
```


<a id="org32ddadf"></a>

### Widgets

```elisp
   `(widget-button-pressed ((,spec (:foreground ,red))))
   `(widget-documentation ((,spec (:foreground ,green))))
   `(widget-single-line-field
     ((,spec (:background ,gray2 :distant-foreground ,bg))))
   `(widget-field
     ((,spec (:background
              ,gray2 :distant-foreground
              ,bg :box `(:line-width -1 :color ,grey1) :extend t))))
```


<a id="org9f0855b"></a>

### End of Custom Faces

In branch devel, this keeps the end of `custom-theme-set-faces` clean. In branch main, we collect all of the tangled blocks here for cleaner looking source code.

```elisp
  (custom-theme-set-faces
   'tangonov
   `(ansi-color-black ((,spec (:foregound ,gray1 :background ,gray1))))
   `(ansi-color-blue ((,spec (:foreground ,blue :background ,blue))))
   `(ansi-color-bright-black ((,spec (:foreground ,gray2 :background ,gray2))))
   `(ansi-color-bright-blue ((,spec (:foreground ,blue :background ,blue))))
   `(ansi-color-bright-cyan ((,spec (:foreground ,cyan :background ,cyan))))
   `(ansi-color-bright-green ((,spec (:foreground ,green :background ,green))))
   `(ansi-color-bright-magenta ((,spec (:foreground ,magenta :background ,magenta))))
   `(ansi-color-bright-red ((,spec (:foreground ,red :background ,red))))
   `(ansi-color-bright-white ((,spec (:foreground "#FFFFFF" :background "#FFFFFF"))))
   `(ansi-color-bright-yellow ((,spec (:foreground ,yellow :background ,yellow))))
   `(ansi-color-cyan ((,spec (:foreground ,cyan :background ,cyan))))
   `(ansi-color-green ((,spec (:foreground ,green :background ,green))))
   `(ansi-color-magenta ((,spec (:foreground ,magenta :background ,magenta))))
   `(ansi-color-red ((,spec (:foreground ,red :background ,red))))
   `(ansi-color-white ((,spec (:foreground ,fg :background ,fg))))
   `(ansi-color-yellow ((,spec (:foreground ,yellow :background ,yellow))))
   `(avy-goto-char-timer-face
     ((,spec (:inherit 'isearch))))
   `(avy-background-face ((,spec (:foreground ,(tangonov-darken bg 0.2)))))
   `(avy-lead-face
     ((,spec (:foreground ,red :weight bold))))
   `(avy-lead-face-0
     ((,spec (:inherit 'avy-lead-face :foreground ,yellow))))
   `(avy-lead-face-1
     ((,spec (:inheri avy-lead-face :foreground ,(tangonov-darken yellow 0.4)))))
   `(avy-lead-face-2
     ((,spec (:inherit 'avy-lead-face :foreground
                       ,(tangonov-darken yellow 0.6)))))
   `(company-echo-common ((,spec (:foreground ,cyan))))
   `(company-tooltip ((,spec (:background ,bg))))
   `(company-tooltip-annotation ((,spec (:foreground ,fg-alt))))
   `(company-tooltip-common ((,spec (:foreground ,cyan))))
   `(company-tooltip-common-selection
     ((,spec (:foreground ,cyan :weight bold))))
   `(company-tooltip-scrollbar-thumb ((,spec (:background ,violet))))
   `(company-tooltip-scrollbar-track ((,spec (:background ,bg-alt))))
   `(company-tooltip-selection ((,spec (:inherit bold :background ,gray1))))
   `(css-proprietary-property ((,spec (:foreground ,orange))))
   `(css-property ((,spec (:foreground ,green))))
   `(css-selector ((,spec (:foreground ,blue))))
   `(custom-button
     ((,spec
       (:foreground ,blue :background ,bg :box '(:line-width 1 :style none)))))
   `(custom-button-unraised
     ((,spec (:foreground ,violet :background
                          ,bg :box '(:line-width 1 :style none)))))
   `(custom-button-pressed-unraised
     ((,spec
       (:foreground ,bg :background
                    ,violet :box '(:line-width 1 :style none)))))
   `(custom-button-pressed
     ((,spec (:foreground ,bg :background
                          ,blue :box '(:line-width 1 :style none)))))
   `(custom-button-mouse
     ((,spec (:foreground ,bg :background ,blue
                          :box '(:line-width 1 :style none)))))
   `(custom-variable-button ((,spec (:foreground ,green :underline t))))
   `(custom-saved
     ((,spec (:foreground ,green :background
                          ,(tangonov-darken green 0.5) :bold bold))))
   `(custom-comment ((,spec (:foreground ,fg))))
   `(custom-comment-tag ((,spec (:foreground ,gray2))))
   `(custom-modified
     ((,spec (:foreground ,blue :background ,(tangonov-darken blue 0.5)))))
   `(custom-variable-tag ((,spec (:foreground ,magenta))))
   `(custom-visibility ((,spec (:foreground ,blue :underline nil))))
   `(custom-group-subtitle ((,spec (:foreground ,red))))
   `(custom-group-tag ((,spec (:foreground ,violet))))
   `(custom-group-tag-1 ((,spec (:foreground ,blue))))
   `(custom-set ((,spec (:foreground ,yellow :background ,bg))))
   `(custom-themed ((,spec (:foreground ,yellow :background ,bg))))
   `(custom-invalid
     ((,spec (:foreground ,red :background ,(tangonov-darken red 0.5)))))
   `(custom-variable-obsolete ((,spec (:foreground ,gray2 :background ,bg))))
   `(custom-state
     ((,spec (:foreground ,green :background ,(tangonov-darken green 0.5)))))
   `(custom-changed ((,spec (:foreground ,blue :background ,bg))))
   `(elfeed-log-debug-level-face ((,spec (:foreground ,gray2))))
   `(elfeed-log-error-level-face ((,spec (:inherit 'error))))
   `(elfeed-log-info-level-face ((,spec (:inherit 'success))))
   `(elfeed-log-warn-level-face ((,spec (:inherit 'warning))))
   `(elfeed-search-date-face ((,spec (:foreground ,violet))))
   `(elfeed-search-feed-face ((,spec (:foreground ,blue))))
   `(elfeed-search-tag-face ((,spec (:foreground ,gray2))))
   `(elfeed-search-title-face ((,spec (:foreground ,gray2))))
   `(elfeed-search-filter-face ((,spec (:foreground ,violet))))
   `(elfeed-search-unread-count-face ((,spec (:foreground ,yellow))))
   `(elfeed-search-unread-title-face ((,spec (:foreground ,fg :weight bold))))
   `(eww-form-checkbox ((,spec (:inherit 'eww-form-file))))
   `(eww-form-file   ((,spec (:inherit 'eww-form-submit :background ,bg-alt))))
   `(eww-form-select ((,spec (:inherit 'eww-form-submit :background ,bg-alt))))
   `(eww-form-submit
     ((,spec (:inherit 'eww-form-text :box
                       `(:line-width 2 :style released-button)
                       :background ,gray1))))
   `(eww-form-text
     ((,spec (:box `(:line-width 1 :color ,gray2)
                   ,bg :foreground ,fg :distant-foreground ,bg))))
   `(eww-form-textarea ((,spec (:inherit 'eww-form-text))))
   `(eww-invalid-certificate ((,spec (:foreground ,red))))
   `(eww-valid-certificate ((,spec (:foreground ,cyan))))
   `(default ((,spec (:background ,bg :foreground ,fg))))
   `(bold ((,spec (:weight bold))))
   `(italic ((,spec (:slant italic))))
   `(bold-italic ((,spec (:weight bold :slant italic))))
   `(underline ((,spec (:underline t))))
   `(shadow ((,spec (:foreground ,gray2))))
   `(link ((,spec (:foreground ,blue :weight bold :underline t))))
   `(link-visited ((,spec (:inherit 'link :foreground ,magenta))))
   `(highlight ((,spec (:background ,gray1 :weight bold))))
   `(match ((,spec (:foreground
                    ,green :background ,(tangonov-darken green 0.5)))))
   `(region ((,spec (:foreground
                     ,cyan :background ,(tangonov-darken cyan 0.5)))))
   `(secondary-selection ((,spec (:background ,gray2 :foreground ,fg))))
   `(lazy-highlight ((,spec
                      (:foreground ,blue :background
                                   ,(tangonov-darken blue 0.5)))))
   `(error ((,spec (:foreground ,red))))
   `(warning ((,spec (:foreground ,yellow))))
   `(success ((,spec (:foreground ,green))))
   `(escape-glyph ((,spec (:foreground ,orange))))
   `(homoglyph ((,spec (:foreground ,orange))))
   `(vertical-border ((,spec (:foreground ,gray1))))
   `(cursor ((,spec (:background ,yellow))))
   `(minibuffer-prompt ((,spec (:foreground ,yellow))))
   `(line-number-current-line ((,spec (:foreground ,cyan :background ,gray1))))
   `(completions-common-part ((,spec (:foreground ,cyan))))
   `(completions-first-difference ((,spec (:foreground ,yellow))))
   `(trailing-whitespace ((,spec (:background ,red))))
   `(whitespace-trailing ((,spec (:background ,red))))
   `(bookmark-face ((,spec (:foreground ,orange))))
   `(tool-bar ((,spec (:foreground ,fg :background ,bg-alt))))
   `(tooltip ((,spec (:foreground ,fg :background ,bg-alt))))
   `(message-header-name ((,spec (:foreground ,green))))
   `(message-header-subject ((,spec (:foreground ,cyan :weight bold))))
   `(message-header-to ((,spec (:foreground ,cyan :weight bold))))
   `(message-header-cc
     ((,spec (:inherit 'message-header-to
                       :foreground ,(tangonov-darken cyan 0.15)))))
   `(message-header-other ((,spec (:foreground ,violet))))
   `(message-header-newsgroups ((,spec (:foreground ,yellow))))
   `(message-header-xheader ((,spec (:foreground ,gray3))))
   `(message-separator ((,spec (:foreground ,gray2))))
   `(message-mml ((,spec (:foreground ,gray2 :slant italic))))
   `(gnus-group-mail-1 ((,spec (:weight bold :foreground ,fg))))
   `(gnus-group-mail-2 ((,spec (:inherit 'gnus-group-mail-1))))
   `(gnus-group-mail-3 ((,spec (:inherit 'gnus-group-mail-1))))
   `(gnus-group-mail-1-empty ((,spec (:foreground ,gray3))))
   `(gnus-group-mail-2-empty ((,spec (:inherit 'gnus-group-mail-1-empty))))
   `(gnus-group-mail-3-empty ((,spec (:inherit 'gnus-group-mail-1-empty))))
   `(gnus-group-news-1 ((,spec (:inherit 'gnus-group-mail-1))))
   `(gnus-group-news-2 ((,spec (:inherit 'gnus-group-news-1))))
   `(gnus-group-news-3 ((,spec (:inherit 'gnus-group-news-1))))
   `(gnus-group-news-4 ((,spec (:inherit 'gnus-group-news-1))))
   `(gnus-group-news-5 ((,spec (:inherit 'gnus-group-news-1))))
   `(gnus-group-news-6 ((,spec (:inherit 'gnus-group-news-1))))
   `(gnus-group-news-1-empty ((,spec (:inherit 'gnus-group-mail-1-empty))))
   `(gnus-group-news-2-empty ((,spec (:inherit 'gnus-group-news-1-empty))))
   `(gnus-group-news-3-empty ((,spec (:inherit 'gnus-group-news-1-empty))))
   `(gnus-group-news-4-empty ((,spec (:inherit 'gnus-group-news-1-empty))))
   `(gnus-group-news-5-empty ((,spec (:inherit 'gnus-group-news-1-empty))))
   `(gnus-group-news-6-empty ((,spec (:inherit 'gnus-group-news-1-empty))))
   `(gnus-group-mail-low ((,spec (:inherit 'gnus-group-mail-1 :weight normal))))
   `(gnus-group-mail-low-empty ((,spec (:inherit 'gnus-group-mail-1-empty))))
   `(gnus-group-news-low
     ((,spec (:inherit 'gnus-group-mail-1 :foreground ,gray3))))
   `(gnus-group-news-low-empty
     ((,spec (:inherit 'gnus-group-news-low :weight normal))))
   `(gnus-header-content ((,spec (:inherit 'message-header-other))))
   `(gnus-header-from ((,spec (:inherit 'message-header-other))))
   `(gnus-header-name ((,spec (:inherit 'message-header-name))))
   `(gnus-header-newsgroups ((,spec (:inherit 'message-header-other))))
   `(gnus-header-subject ((,spec (:inherit 'message-header-subject))))
   `(gnus-summary-cancelled ((,spec (:foreground ,red :strike-through t))))
   `(gnus-summary-high-ancient
     ((,spec (:foreground ,(tangonov-lighten gray3 0.2) :inherit 'italic))))
   `(gnus-summary-high-read
     ((,spec (:foreground ,(tangonov-lighten fg 0.2)))))
   `(gnus-summary-high-ticked
     ((,spec (:foreground ,(tangonov-lighten magenta 0.2)))))
   `(gnus-summary-high-unread
     ((,spec (:foreground ,(tangonov-lighten green 0.2)))))
   `(gnus-summary-low-ancient
     ((,spec (:foreground ,(tangonov-darken gray3 0.2) :inherit 'italic))))
   `(gnus-summary-low-read ((,spec (:foreground ,(tangonov-darken fg 0.2)))))
   `(gnus-summary-low-ticked
     ((,spec (:foreground ,(tangonov-darken magenta 0.2)))))
   `(gnus-summary-low-unread
     ((,spec (:foreground ,(tangonov-darken green 0.2)))))
   `(gnus-summary-normal-ancient
     ((,spec (:foreground ,gray3 :inherit 'italic))))
   `(gnus-summary-normal-read ((,spec (:foreground ,fg))))
   `(gnus-summary-normal-ticked ((,spec (:foreground ,magenta))))
   `(gnus-summary-normal-unread ((,spec (:foreground ,green :inherit 'bold))))
   `(gnus-summary-selected ((,spec (:foreground ,blue :weight bold))))
   `(gnus-cite-1 ((,spec (:foreground ,violet))))
   `(gnus-cite-2 ((,spec (:foreground ,yellow))))
   `(gnus-cite-3 ((,spec (:foreground ,magenta))))
   `(gnus-cite-4 ((,spec (:foreground ,green))))
   `(gnus-cite-5 ((,spec (:foreground ,green))))
   `(gnus-cite-6 ((,spec (:foreground ,green))))
   `(gnus-cite-7 ((,spec (:foreground ,magenta))))
   `(gnus-cite-8 ((,spec (:foreground ,magenta))))
   `(gnus-cite-9 ((,spec (:foreground ,magenta))))
   `(gnus-cite-10 ((,spec (:foreground ,yellow))))
   `(gnus-cite-11 ((,spec (:foreground ,yellow))))
   `(gnus-signature ((,spec (:foreground ,yellow))))
   `(gnus-x-face ((,spec (:background ,gray3 :foreground ,fg))))
   `(notmuch-crypto-decryption ((,spec (:foreground ,magenta))))
   `(notmuch-crypto-signature-bad ((,spec (:foreground ,red))))
   `(notmuch-crypto-signature-good ((,spec (:foreground ,green))))
   `(notmuch-crypto-signature-good-key ((,spec (:foreground ,orange))))
   `(notmuch-crypto-signature-unknown ((,spec (:foreground ,red))))
   `(notmuch-message-summary-face
     ((,spec (:background ,bg-alt :overline ,gray2))))
   `(notmuch-search-count ((,spec (:foreground ,gray2))))
   `(notmuch-search-date ((,spec (:foreground ,orange))))
   `(notmuch-search-flagged-face
     ((,spec (:foreground ,(tangonov-darken red 0.5)))))
   `(notmuch-search-matching-authors ((,spec (:foreground ,blue))))
   `(notmuch-search-non-matching-authors ((,spec (:foreground ,fg))))
   `(notmuch-search-subject ((,spec (:foreground ,fg))))
   `(notmuch-search-unread-face ((,spec (:weight bold))))
   `(notmuch-tag-added ((,spec (:foreground ,green :weight normal))))
   `(notmuch-tag-deleted ((,spec (:foreground ,red :weight normal))))
   `(notmuch-tag-face ((,spec (:foreground ,yellow :weight normal))))
   `(notmuch-tag-flagged ((,spec (:foreground ,yellow :weight normal))))
   `(notmuch-tag-unread ((,spec (:foreground ,yellow :weight normal))))
   `(notmuch-tree-match-author-face ((,spec (:foreground ,blue :weight bold))))
   `(notmuch-tree-match-date-face ((,spec (:foreground ,orange :weight bold))))
   `(notmuch-tree-match-face ((,spec (:foreground ,fg))))
   `(notmuch-tree-match-subject-face ((,spec (:foreground ,fg))))
   `(notmuch-tree-match-tag-face ((,spec (:foreground ,yellow))))
   `(notmuch-tree-match-tree-face ((,spec (:foreground ,gray2))))
   `(notmuch-tree-no-match-author-face ((,spec (:foreground ,blue))))
   `(notmuch-tree-no-match-date-face ((,spec (:foreground ,orange))))
   `(notmuch-tree-no-match-face ((,spec (:foreground ,gray3))))
   `(notmuch-tree-no-match-subject-face ((,spec (:foreground ,gray3))))
   `(notmuch-tree-no-match-tag-face ((,spec (:foreground ,yellow))))
   `(notmuch-tree-no-match-tree-face ((,spec (:foreground ,yellow))))
   `(notmuch-wash-cited-text ((,spec (:foreground ,gray1))))
   `(notmuch-wash-toggle-button ((,spec (:foreground ,fg))))
   `(erc-button ((,spec (:weight bold :underline t))))
   `(erc-default-face ((,spec (:inherit 'default))))
   `(erc-action-face ((,spec (:weight bold))))
   `(erc-command-indicator-face ((,spec (:weight bold))))
   `(erc-direct-msg-face ((,spec (:foreground ,magenta))))
   `(erc-error-face ((,spec (:inherit 'error))))
   `(erc-header-line
     ((,spec (:background ,(tangonov-darken bg-alt 0.15) :foreground ,cyan))))
   `(erc-input-face ((,spec (:foreground ,green))))
   `(erc-current-nick-face ((,spec (:foreground ,green :weight bold))))
   `(erc-timestamp-face ((,spec (:foreground ,blue :weight bold))))
   `(erc-nick-default-face ((,spec (:weight bold))))
   `(erc-nick-msg-face ((,spec (:foreground ,magenta))))
   `(erc-nick-prefix-face ((,spec (:inherit 'erc-nick-default-face))))
   `(erc-my-nick-face ((,spec (:foreground ,green :weight bold))))
   `(erc-my-nick-prefix-face ((,spec (:inherit 'erc-my-nick-face))))
   `(erc-notice-face ((,spec (:foreground ,gray2))))
   `(erc-prompt-face ((,spec (:foreground ,cyan :weight bold))))
   `(evil-ex-info ((,spec (:foreground ,red :slant italic))))
   `(evil-ex-search
     ((,spec (:background ,gray1 :foreground ,cyan :weight bold))))
   `(evil-ex-substitute-matches
     ((,spec (:background ,gray1 :foreground
                          ,red :weight bold :strike-through t))))
   `(evil-ex-substitute-replacement
     ((,spec (:background ,gray1 :foreground ,green :weight bold))))
   `(evil-search-highlight-persist-highlight-face
     ((,spec (:inherit 'lazy-highlight))))
   `(evil-mc-cursor-default-face
     ((,spec (:background ,magenta :foreground ,gray1 :inverse-video nil))))
   `(evil-mc-region-face ((,spec (:inherit 'region))))
   `(evil-mc-cursor-bar-face
     ((,spec (:height 1 :background ,magenta :foreground ,gray1))))
   `(evil-mc-cursor-hbar-face ((,spec (:underline `(:color ,cyan)))))
   `(evil-snipe-first-match-face
     ((,spec (:foreground ,blue :background
                          ,(tangonov-darken blue 0.5) :weight bold))))
   `(evil-snipe-matches-face
     ((,spec (:foreground highlight :underline t :weight bold))))
   `(evil-goggles-delete-face
     ((,spec (:foreground ,(tangonov-darken red 0.5) :background ,red))))
   `(evil-goggles-paste-face
     ((,spec (:foreground ,(tangonov-darken green 0.5) :background ,green))))
   `(evil-goggles-undo-redo-add-face ((,spec (:inherit 'evil-goggles-paste-face))))
   `(evil-goggles-undo-redo-remove-face ((,spec (:inherit 'evil-goggles-delete-face))))
   `(evil-goggles-record-macro-face
     ((,spec (:foreground ,(tangonov-darken yellow 0.5) :background ,yellow))))
   ;; Font Lock
   `(font-lock-warning-face ((,spec (:inherit 'warning))))
   `(font-lock-function-name-face ((,spec (:foreground ,blue))))
   `(font-lock-variable-name-face ((,spec (:foreground ,yellow))))
   `(font-lock-keyword-face ((,spec (:foreground ,cyan))))
   `(font-lock-comment-face ((,spec (:foreground ,gray2))))
   `(font-lock-type-face ((,spec (:foreground ,magenta))))
   `(font-lock-constant-face ((,spec (:foreground ,orange))))
   `(font-lock-builtin-face ((,spec (:foreground ,cyan))))
   `(font-lock-string-face ((,spec (:foreground ,green))))
   `(font-lock-doc-face ((,spec (:foreground ,gray2))))
   `(font-lock-negation-char-face ((,spec (:foreground ,orange))))
   `(goggles-changed ((,spec (:background ,cyan))))
   `(goggles-added ((,spec (:background ,green))))
   `(goggles-removed ((,spec (:background ,red))))
   `(hydra-face-red ((,spec (:foreground ,red :weight bold))))
   `(hydra-face-blue ((,spec (:foreground ,blue :weight bold))))
   `(hydra-face-amaranth ((,spec (:foreground ,magenta :weight bold))))
   `(hydra-face-pink ((,spec (:foreground ,violet :weight bold))))
   `(hydra-face-teal ((,spec (:foreground ,teal :weight bold))))
   `(inf-ruby-result-overlay-face
     ((,spec (:foreground ,cyan :background
                          ,bg-alt :box (:line-width 1 :color ,cyan)))))
   `(isearch ((,spec (:inherit 'match :weight bold))))
   `(isearch-fail ((,spec (:background ,red :foreground ,gray1 :weight bold))))
   `(keycast-key
     ((,spec (:weight bold
                      :background ,green
                      :foreground ,bg))))
   `(keycast-command ((,spec (:foreground ,green :weight bold))))
   `(flymake-error ((,spec (:underline (:style wave :color ,red)))))
   `(flymake-note ((,spec (:underline (:style wave :color ,green)))))
   `(flymake-warning ((,spec (:underline (:style wave :color ,orange)))))
   `(flycheck-error ((,spec (:underline (:style wave :color ,red)))))
   `(flycheck-warning ((,spec (:underline (:style wave :color ,yellow)))))
   `(flycheck-info ((,spec (:underline (:style wave :color ,green)))))
   `(flycheck-fringe-error ((,spec (:inherit 'fringe :foreground ,red))))
   `(flycheck-fringe-warning ((,spec (:inherit 'fringe :foreground ,yellow))))
   `(flycheck-fringe-info ((,spec (:inherit 'fringe :foreground ,green))))
   `(flycheck-posframe-face ((,spec (:inherit 'default))))
   `(flycheck-posframe-background-face ((,spec (:background ,bg-alt))))
   `(flycheck-posframe-error-face
     ((,spec (:inherit 'flycheck-posframe-face :foreground ,red))))
   `(flycheck-posframe-info-face
     ((,spec (:inherit 'flycheck-posframe-face :foreground ,fg))))
   `(flycheck-posframe-warning-face
     ((,spec (:inherit 'flycheck-posframe-face :foreground ,yellow))))
   `(flyspell-incorrect
     ((,spec (:underline (:style wave :color ,red) :inherit 'unspecified))))
   `(flyspell-duplicate
     ((,spec (:underline (:style wave :color ,yellow) :inherit 'unspecified))))
   `(eglot-highlight-symbol-face ((,spec (:weight bold :background ,gray1))))
   `(eldoc-box-border ((,spec (:background ,fg-alt))))
   `(mode-line
     ((,spec (:foreground ,fg-alt :background ,bg-alt :box
                          (:line-width (2 . 2) :color ,bg-alt)))))
   `(mode-line-inactive
     ((,spec (:inherit 'mode-line :foreground ,gray2 :background ,bg))))
   `(mode-line-highlight ((,spec (:box (:line-width (2 . 2) :color ,magenta)))))
   `(mode-line-buffer-id ((,spec (:foreground ,fg :weight bold))))
   `(org-block ((,spec (:background ,bg-alt))))
   `(org-block-background ((,spec (:background ,bg-alt))))
   `(org-block-begin-line ((,spec (:foreground ,gray2 :background ,bg))))
   `(org-level-1 ((,spec (:inherit bold :foreground ,green))))
   `(org-level-2 ((,spec (:inherit bold :foreground ,yellow))))
   `(org-level-3 ((,spec (:inherit bold :foreground ,red))))
   `(org-level-4 ((,spec (:inherit bold :foreground ,cyan))))
   `(org-level-5 ((,spec (:inherit bold :foreground ,blue))))
   `(org-level-6 ((,spec (:inherit bold :foreground ,magenta))))
   `(org-level-7 ((,spec (:inherit bold :foreground ,teal))))
   `(org-level-8 ((,spec (:inherit bold :foreground ,violet))))
   `(org-headline-done ((,spec (:foreground ,gray2))))
   `(org-table ((,spec (:foreground ,magenta))))
   `(org-todo ((,spec (:foreground ,orange))))
   `(org-done ((,spec (:foreground ,gray2))))
   `(org-drawer ((,spec (:foreground ,gray2))))
   `(org-meta-line ((,spec (:foreground ,gray2))))
   `(org-special-keyword ((,spec (:foreground ,gray3))))
   `(org-property-value ((,spec (:foreground ,red))))
   `(org-tag ((,spec (:foreground ,fg-alt))))
   `(org-verbatim ((,spec (:foreground ,green))))
   `(org-code ((,spec (:foreground ,orange :background ,bg-alt))))
   `(org-document-info-keyword ((,spec (:foreground ,red))))
   `(org-document-info ((,spec (:foreground ,fg-alt))))
   `(org-document-title ((,spec (:foreground ,yellow))))
   `(org-date ((,spec (:foreground ,yellow))))
   `(org-checkbox ((,spec (:foreground ,orange))))
   `(org-checkbox-statistics-todo ((,spec (:inherit 'org-checkbox))))
   `(org-checkbox-statistics-done ((,spec (:inherit 'org-done))))
   `(org-agenda-done ((,spec (:inherit 'org-done))))
   `(org-agenda-clocking
     ((,spec (:background ,(tangonov-darken cyan 0.5) :extend t))))
   `(org-time-grid ((,spec (:foreground ,gray2))))
   `(org-imminent-deadline ((,spec (:foreground ,yellow))))
   `(org-upcoming-deadline ((,spec (:foreground ,teal))))
   `(org-agenda-dimmed-todo-face ((,spec (:foreground ,gray3))))
   `(org-habit-clear-face ((,spec (:weight bold :background ,gray2))))
   `(org-habit-clear-future-face ((,spec (:weight bold :background ,gray3))))
   `(org-habit-ready-face
     ((,spec (:weight bold :background ,(tangonov-darken blue 0.5)))))
   `(org-habit-ready-future-face
     ((,spec (:weight bold :background ,(tangonov-darken blue 0.3)))))
   `(org-habit-alert-face
     ((,spec (:weight bold :background ,(tangonov-darken yellow 0.5)))))
   `(org-habit-alert-future-face
     ((,spec (:weight bold :background ,(tangonov-darken yellow 0.3)))))
   `(org-habit-overdue-face
     ((,spec (:weight bold :background ,(tangonov-darken red 0.5)))))
   `(org-habit-overdue-future-face
     ((,spec (:weight bold :background ,(tangonov-darken red 0.3)))))
   `(org-journal-highlight ((,spec (:foreground ,violet))))
   `(org-journal-calendar-entry-face
     ((,spec (:foreground ,magenta :slant italic))))
   `(org-journal-calendar-scheduled-face
     ((,spec (:foreground ,red :slant italic))))
   `(org-pomodoro-mode-line ((,spec (:foreground ,fg))))
   `(org-pomodoro-mode-line-overtime
     ((,spec (:foreground ,yellow :weight bold))))
   `(org-mode-line-clock ((,spec (:foreground ,fg))))
   `(org-mode-line-clock-overrun ((,spec (:inherit error))))
   `(org-ref-acronym-face ((,spec (:foreground ,violet))))
   `(org-ref-cite-face
     ((,spec (:foreground ,yellow :weight light :underline t))))
   `(org-ref-glossary-face ((,spec (:foreground ,magenta))))
   `(org-ref-label-face ((,spec (:foreground ,blue))))
   `(org-ref-ref-face ((,spec (:inherit 'link :foreground ,teal))))
   `(rainbow-delimiters-depth-1-face ((,spec (:foreground ,magenta))))
   `(rainbow-delimiters-depth-2-face ((,spec (:foreground ,orange))))
   `(rainbow-delimiters-depth-3-face ((,spec (:foreground ,green))))
   `(rainbow-delimiters-depth-4-face ((,spec (:foreground ,cyan))))
   `(rainbow-delimiters-depth-5-face ((,spec (:foreground ,violet))))
   `(rainbow-delimiters-depth-6-face ((,spec (:foreground ,yellow))))
   `(rainbow-delimiters-depth-7-face ((,spec (:foreground ,blue))))
   `(rainbow-delimiters-depth-8-face ((,spec (:foreground ,teal))))
   `(rainbow-delimiters-depth-9-face ((,spec (:foreground ,red))))
   `(rjsx-tag ((,spec (:foreground ,red))))
   `(rjsx-attr ((,spec (:foreground ,yellow :slant italic :weight medium))))
   `(rjsx-tag-bracket-face ((,spec (:foreground ,cyan))))
   `(eshell-prompt ((,spec (:foreground ,magenta :weight bold))))
   `(eshell-ls-archive ((,spec (:foreground ,gray2))))
   `(eshell-ls-backup ((,spec (:foreground ,yellow))))
   `(eshell-ls-clutter ((,spec (:foreground ,red))))
   `(eshell-ls-directory ((,spec (:foreground ,blue))))
   `(eshell-ls-executable ((,spec (:foreground ,green))))
   `(eshell-ls-missing ((,spec (:foreground ,red))))
   `(eshell-ls-product ((,spec (:foreground ,orange))))
   `(eshell-ls-readonly ((,spec (:foreground ,orange))))
   `(eshell-ls-special ((,spec (:foreground ,violet))))
   `(eshell-ls-symlink ((,spec (:foreground ,cyan))))
   `(eshell-ls-unreadable ((,spec (:foreground ,gray3))))
   `(vterm-color-black
     ((,spec (:background ,gray1 :foreground ,(tangonov-lighten gray1 0.2)))))
   `(vterm-color-red
     ((,spec (:background ,red :foreground ,(tangonov-lighten red 0.2)))))
   `(vterm-color-green
     ((,spec (:background ,green :foreground ,(tangonov-lighten green 0.2)))))
   `(vterm-color-yellow
     ((,spec (:background ,yellow :foreground ,(tangonov-lighten yellow 0.2)))))
   `(vterm-color-blue
     ((,spec (:background ,blue :foreground ,(tangonov-lighten blue 0.2)))))
   `(vterm-color-magenta
     ((,spec (:background ,magenta :foreground
                          ,(tangonov-lighten violet 0.2)))))
   `(vterm-color-cyan
     ((,spec (:background ,cyan :foreground ,(tangonov-lighten cyan 0.2)))))
   `(vterm-color-white ((,spec (:background ,fg :foreground ,gray3))))
   `(tab-line ((,spec (:background ,bg-alt))))
   `(tab-line-tab ((,spec (:background ,bg-alt :foreground ,fg-alt))))
   `(tab-line-tab-inactive
     ((,spec (:inherit 'tab-line-tab
                       :background ,bg-alt :foreground ,gray2))))
   `(tab-line-tab-inactive-alternate
     ((,spec (:inherit 'tab-line-tab-inactive))))
   `(tab-line-tab-current ((,spec (:background ,bg-alt :foreground ,fg))))
   `(tab-line-highlight ((,spec (:inherit 'tab-line-tab))))
   `(tab-line-close-highlight ((,spec (:foreground ,cyan))))
   `(tab-bar ((,spec (:inherit tab-line))))
   `(tab-bar-tab ((,spec (:inherit tab-line-tab))))
   `(tab-bar-tab-inactive ((,spec (:inherit tab-line-tab-inactive))))
   `(typescript-jsdoc-tag ((,spec (:foreground ,magenta))))
   `(typescript-jsdoc-type ((,spec (:foreground ,gray3))))
   `(typescript-jsdoc-value ((,spec (:foreground ,cyan))))
   `(diff-added ((,spec
                  (:foreground ,green :background
                               ,(tangonov-darken green 0.5)))))
   `(diff-changed
     ((,spec (:foreground ,blue :background ,(tangonov-darken blue 0.5)))))
   `(diff-context ((,spec (:foreground ,gray3))))
   `(diff-removed
     ((,spec (:foreground ,red :background ,(tangonov-darken red 0.5)))))
   `(diff-header ((,spec (:foreground ,cyan))))
   `(diff-file-header ((,spec (:foreground ,blue :background ,bg))))
   `(diff-hunk-header ((,spec (:foreground ,violet))))
   `(diff-refine-added ((,spec (:inherit 'diff-added :inverse-video t))))
   `(diff-refine-changed ((,spec (:inherit 'diff-changed :inverse-video t))))
   `(diff-refine-removed ((,spec (:inherit 'diff-removed :inverse-video t))))
   `(diff-hl-change ((,spec (:background ,blue :foreground ,blue))))
   `(diff-hl-delete ((,spec (:background ,red :foreground ,red))))
   `(diff-hl-insert ((,spec (:background ,green :foreground ,green))))
   `(ediff-fine-diff-A ((,spec
                         (:background
                          ,(tangonov-blend cyan bg 0.7) :weight bold :extend))))
   `(ediff-fine-diff-B ((,spec (:inherit 'ediff-fine-diff-A))))
   `(ediff-fine-diff-C ((,spec (:inherit 'ediff-fine-diff-A))))
   `(ediff-current-diff-A
     ((,spec (:background ,(tangonov-blend cyan bg 0.3) :extend t))))
   `(ediff-current-diff-B ((,spec (:inherit 'ediff-current-diff-A))))
   `(ediff-current-diff-C ((,spec (:inherit 'ediff-current-diff-A))))
   `(ediff-even-diff-A ((,spec (:inherit 'hl-line))))
   `(ediff-even-diff-B ((,spec (:inherit 'ediff))))
   `(ediff-even-diff-C ((,spec (:inherit 'ediff-even-diff-A))))
   `(ediff-odd-diff-A ((,spec (:inherit 'ediff-even-diff-A))))
   `(ediff-odd-diff-B ((,spec (:inherit 'ediff-odd-diff-A))))
   `(ediff-odd-diff-C ((,spec (:inherit 'ediff-odd-diff-A))))
   `(magit-bisect-bad ((,spec (:foreground ,red))))
   `(magit-bisect-good ((,spec (:foreground ,green))))
   `(magit-bisect-skip ((,spec (:foreground ,orange))))
   `(magit-blame-hash ((,spec (:foreground ,cyan))))
   `(magit-blame-date ((,spec (:foreground ,red))))
   `(magit-blame-heading
     ((,spec (:foreground ,orange :background ,gray3 :extend t))))
   `(magit-branch-current ((,spec (:foreground ,blue))))
   `(magit-branch-local ((,spec (:foreground ,cyan))))
   `(magit-branch-remote ((,spec (:foreground ,green))))
   `(magit-cherry-equivalent ((,spec (:foreground ,violet))))
   `(magit-cherry-unmatched ((,spec (:foreground ,cyan))))
   `(magit-diff-added
     ((,spec (:foreground ,(tangonov-darken green 0.2) :background
                          ,(tangonov-blend green bg 0.1) :extend t))))
   `(magit-diff-added-highlight
     ((,spec (:foreground ,green :background
                          ,(tangonov-blend green bg 0.2)
                          :weight bold :extend t))))
   `(magit-diff-base
     ((,spec (:foreground ,(tangonov-darken orange 0.2) :background
                          ,(tangonov-blend orange bg 0.1) :extend t))))
   `(magit-diff-base-highlight
     ((,spec (:foreground ,orange :background
                          ,(tangonov-blend orange bg 0.2) :weight
                          bold :extend t))))
   `(magit-diff-context
     ((,spec (:foreground ,(tangonov-darken fg 0.4) :background
                          ,bg :extend t))))
   `(magit-diff-context-highlight
     ((,spec (:foreground ,fg :background ,bg-alt :extend t))))
   `(magit-diff-file-heading
     ((,spec (:foreground ,fg :weight bold :extend t))))
   `(magit-diff-file-heading-selection
     ((,spec (:foreground ,magenta :background
                          ,(tangonov-darken blue 0.5) :weight bold :extend t))))
   `(magit-diff-hunk-heading
     ((,spec (:foreground ,bg :background
                          ,(tangonov-blend violet bg 0.3) :extend t))))
   `(magit-diff-hunk-heading-highlight
     ((,spec (:foreground ,bg :background ,violet :weight bold :extend t))))
   `(magit-diff-lines-heading
     ((,spec (:foreground ,yellow :background ,red :extend t :extend t))))
   `(magit-diff-removed
     ((,spec (:foreground ,(tangonov-darken red 0.2) :background
                          ,(tangonov-blend red bg 0.1) :extend t))))
   `(magit-diff-removed-highlight
     ((,spec (:foreground ,red :background
                          ,(tangonov-blend red bg 0.2)
                          :weight bold :extend t))))
   `(magit-diffstat-added ((,spec (:foreground ,green))))
   `(magit-diffstat-removed ((,spec (:foreground ,red))))
   `(magit-dimmed ((,spec (:foreground ,gray2))))
   `(magit-hash ((,spec (:foreground ,gray2))))
   `(magit-header-line
     ((,spec (:background ,bg-alt :foreground ,yellow :weight bold))))
   `(magit-filename ((,spec (:foreground ,violet))))
   `(magit-log-author ((,spec (:foreground ,orange))))
   `(magit-log-date ((,spec (:foreground ,blue))))
   `(magit-log-graph ((,spec (:foreground ,gray2))))
   `(magit-process-ng ((,spec (:inherit 'error))))
   `(magit-process-ok ((,spec (:inherit 'success))))
   `(magit-reflog-amend ((,spec (:foreground ,magenta))))
   `(magit-reflog-checkout ((,spec (:foreground ,blue))))
   `(magit-reflog-cherry-pick ((,spec (:foreground ,green))))
   `(magit-reflog-commit ((,spec (:foreground ,green))))
   `(magit-reflog-merge ((,spec (:foreground ,green))))
   `(magit-reflog-other ((,spec (:foreground ,cyan))))
   `(magit-reflog-rebase ((,spec (:foreground ,magenta))))
   `(magit-reflog-remote ((,spec (:foreground ,cyan))))
   `(magit-reflog-reset ((,spec (:inherit 'error))))
   `(magit-refname ((,spec (:foreground ,gray2))))
   `(magit-section-heading
     ((,spec (:foreground ,blue :weight bold :extend t))))
   `(magit-section-heading-selection
     ((,spec (:foreground ,orange :weight bold :extend t))))
   `(magit-section-highlight ((,spec (:inherit 'hl-line))))
   `(magit-section-secondary-heading
     ((,spec (:foreground ,violet :weight bold :extend t))))
   `(magit-sequence-drop ((,spec (:foreground ,red))))
   `(magit-sequence-head ((,spec (:foreground ,blue))))
   `(magit-sequence-part ((,spec (:foreground ,orange))))
   `(magit-sequence-stop ((,spec (:foreground ,green))))
   `(magit-signature-bad ((,spec (:inherit 'error))))
   `(magit-signature-error ((,spec (:inherit 'error))))
   `(magit-signature-expired ((,spec (:foreground ,orange))))
   `(magit-signature-good ((,spec (:inherit 'success))))
   `(magit-signature-revoked ((,spec (:foreground ,magenta))))
   `(magit-signature-untrusted ((,spec (:foreground ,yellow))))
   `(magit-tag ((,spec (:foreground ,yellow))))
   `(web-mode-html-tag-face ((,spec (:foreground ,red))))
   `(web-mode-html-attr-equal-face ((,spec (:foreground ,cyan))))
   `(widget-button-pressed ((,spec (:foreground ,red))))
   `(widget-documentation ((,spec (:foreground ,green))))
   `(widget-single-line-field
     ((,spec (:background ,gray2 :distant-foreground ,bg))))
   `(widget-field
     ((,spec (:background
              ,gray2 :distant-foreground
              ,bg :box `(:line-width -1 :color ,grey1) :extend t))))))
```


<a id="orgdb4c000"></a>

## Add Theme to Custom Load Path

It's not enough to simply provide a theme. We must set up an autoload that will put the theme into the `custom-theme-load-path`. I suppose users would have to adjust their load path without this.

```elisp
;;;###autoload
(when (and (bound-and-true-p custom-theme-load-path)
           load-file-name)
  (add-to-list 'custom-theme-load-path
               (file-name-as-directory (file-name-directory load-file-name))))
```


<a id="orgd721327"></a>

## Provide the Theme

This file needs to be symbolically represented as a theme, not a "package". So, we must `(provide-theme)`.

```elisp
(provide-theme 'tangonov)
```


<a id="orgebea302"></a>

## Theme Footer

Provide the theme and mark the end of the file.

```elisp
;;; tangonov-theme.el ends here
```


<a id="org2eda3e0"></a>

## Contributing

Thanks for your interest in this project. Development is done on the `devel` branch. If you would like to contribute, please:

1.  Fork this repository.
2.  Clone it locally and make a **new branch** from branch `devel` for your feature.
3.  Make your changes in the new branch, push them & submit your pull request.

**Note:** This theme is developed using literate programming. This is to say, the document you are viewing **is** the theme. If you're not sure what this means, look into literate programming with org-mode. If you are wanting to make a contribution but do not know/do not want to know how to use this document, I will try to work with you.


<a id="org46e2ab2"></a>

## License

This program is free software; you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

![img](images/gplv3.png)

```markdown
### GNU GENERAL PUBLIC LICENSE

Version 3, 29 June 2007

Copyright (C) 2007 Free Software Foundation, Inc.
<https://fsf.org/>

Everyone is permitted to copy and distribute verbatim copies of this
license document, but changing it is not allowed.

### Preamble

The GNU General Public License is a free, copyleft license for
software and other kinds of works.

The licenses for most software and other practical works are designed
to take away your freedom to share and change the works. By contrast,
the GNU General Public License is intended to guarantee your freedom
to share and change all versions of a program--to make sure it remains
free software for all its users. We, the Free Software Foundation, use
the GNU General Public License for most of our software; it applies
also to any other work released this way by its authors. You can apply
it to your programs, too.

When we speak of free software, we are referring to freedom, not
price. Our General Public Licenses are designed to make sure that you
have the freedom to distribute copies of free software (and charge for
them if you wish), that you receive source code or can get it if you
want it, that you can change the software or use pieces of it in new
free programs, and that you know you can do these things.

To protect your rights, we need to prevent others from denying you
these rights or asking you to surrender the rights. Therefore, you
have certain responsibilities if you distribute copies of the
software, or if you modify it: responsibilities to respect the freedom
of others.

For example, if you distribute copies of such a program, whether
gratis or for a fee, you must pass on to the recipients the same
freedoms that you received. You must make sure that they, too, receive
or can get the source code. And you must show them these terms so they
know their rights.

Developers that use the GNU GPL protect your rights with two steps:
(1) assert copyright on the software, and (2) offer you this License
giving you legal permission to copy, distribute and/or modify it.

For the developers' and authors' protection, the GPL clearly explains
that there is no warranty for this free software. For both users' and
authors' sake, the GPL requires that modified versions be marked as
changed, so that their problems will not be attributed erroneously to
authors of previous versions.

Some devices are designed to deny users access to install or run
modified versions of the software inside them, although the
manufacturer can do so. This is fundamentally incompatible with the
aim of protecting users' freedom to change the software. The
systematic pattern of such abuse occurs in the area of products for
individuals to use, which is precisely where it is most unacceptable.
Therefore, we have designed this version of the GPL to prohibit the
practice for those products. If such problems arise substantially in
other domains, we stand ready to extend this provision to those
domains in future versions of the GPL, as needed to protect the
freedom of users.

Finally, every program is threatened constantly by software patents.
States should not allow patents to restrict development and use of
software on general-purpose computers, but in those that do, we wish
to avoid the special danger that patents applied to a free program
could make it effectively proprietary. To prevent this, the GPL
assures that patents cannot be used to render the program non-free.

The precise terms and conditions for copying, distribution and
modification follow.

### TERMS AND CONDITIONS

#### 0. Definitions.

"This License" refers to version 3 of the GNU General Public License.

"Copyright" also means copyright-like laws that apply to other kinds
of works, such as semiconductor masks.

"The Program" refers to any copyrightable work licensed under this
License. Each licensee is addressed as "you". "Licensees" and
"recipients" may be individuals or organizations.

To "modify" a work means to copy from or adapt all or part of the work
in a fashion requiring copyright permission, other than the making of
an exact copy. The resulting work is called a "modified version" of
the earlier work or a work "based on" the earlier work.

A "covered work" means either the unmodified Program or a work based
on the Program.

To "propagate" a work means to do anything with it that, without
permission, would make you directly or secondarily liable for
infringement under applicable copyright law, except executing it on a
computer or modifying a private copy. Propagation includes copying,
distribution (with or without modification), making available to the
public, and in some countries other activities as well.

To "convey" a work means any kind of propagation that enables other
parties to make or receive copies. Mere interaction with a user
through a computer network, with no transfer of a copy, is not
conveying.

An interactive user interface displays "Appropriate Legal Notices" to
the extent that it includes a convenient and prominently visible
feature that (1) displays an appropriate copyright notice, and (2)
tells the user that there is no warranty for the work (except to the
extent that warranties are provided), that licensees may convey the
work under this License, and how to view a copy of this License. If
the interface presents a list of user commands or options, such as a
menu, a prominent item in the list meets this criterion.

#### 1. Source Code.

The "source code" for a work means the preferred form of the work for
making modifications to it. "Object code" means any non-source form of
a work.

A "Standard Interface" means an interface that either is an official
standard defined by a recognized standards body, or, in the case of
interfaces specified for a particular programming language, one that
is widely used among developers working in that language.

The "System Libraries" of an executable work include anything, other
than the work as a whole, that (a) is included in the normal form of
packaging a Major Component, but which is not part of that Major
Component, and (b) serves only to enable use of the work with that
Major Component, or to implement a Standard Interface for which an
implementation is available to the public in source code form. A
"Major Component", in this context, means a major essential component
(kernel, window system, and so on) of the specific operating system
(if any) on which the executable work runs, or a compiler used to
produce the work, or an object code interpreter used to run it.

The "Corresponding Source" for a work in object code form means all
the source code needed to generate, install, and (for an executable
work) run the object code and to modify the work, including scripts to
control those activities. However, it does not include the work's
System Libraries, or general-purpose tools or generally available free
programs which are used unmodified in performing those activities but
which are not part of the work. For example, Corresponding Source
includes interface definition files associated with source files for
the work, and the source code for shared libraries and dynamically
linked subprograms that the work is specifically designed to require,
such as by intimate data communication or control flow between those
subprograms and other parts of the work.

The Corresponding Source need not include anything that users can
regenerate automatically from other parts of the Corresponding Source.

The Corresponding Source for a work in source code form is that same
work.

#### 2. Basic Permissions.

All rights granted under this License are granted for the term of
copyright on the Program, and are irrevocable provided the stated
conditions are met. This License explicitly affirms your unlimited
permission to run the unmodified Program. The output from running a
covered work is covered by this License only if the output, given its
content, constitutes a covered work. This License acknowledges your
rights of fair use or other equivalent, as provided by copyright law.

You may make, run and propagate covered works that you do not convey,
without conditions so long as your license otherwise remains in force.
You may convey covered works to others for the sole purpose of having
them make modifications exclusively for you, or provide you with
facilities for running those works, provided that you comply with the
terms of this License in conveying all material for which you do not
control copyright. Those thus making or running the covered works for
you must do so exclusively on your behalf, under your direction and
control, on terms that prohibit them from making any copies of your
copyrighted material outside their relationship with you.

Conveying under any other circumstances is permitted solely under the
conditions stated below. Sublicensing is not allowed; section 10 makes
it unnecessary.

#### 3. Protecting Users' Legal Rights From Anti-Circumvention Law.

No covered work shall be deemed part of an effective technological
measure under any applicable law fulfilling obligations under article
11 of the WIPO copyright treaty adopted on 20 December 1996, or
similar laws prohibiting or restricting circumvention of such
measures.

When you convey a covered work, you waive any legal power to forbid
circumvention of technological measures to the extent such
circumvention is effected by exercising rights under this License with
respect to the covered work, and you disclaim any intention to limit
operation or modification of the work as a means of enforcing, against
the work's users, your or third parties' legal rights to forbid
circumvention of technological measures.

#### 4. Conveying Verbatim Copies.

You may convey verbatim copies of the Program's source code as you
receive it, in any medium, provided that you conspicuously and
appropriately publish on each copy an appropriate copyright notice;
keep intact all notices stating that this License and any
non-permissive terms added in accord with section 7 apply to the code;
keep intact all notices of the absence of any warranty; and give all
recipients a copy of this License along with the Program.

You may charge any price or no price for each copy that you convey,
and you may offer support or warranty protection for a fee.

#### 5. Conveying Modified Source Versions.

You may convey a work based on the Program, or the modifications to
produce it from the Program, in the form of source code under the
terms of section 4, provided that you also meet all of these
conditions:

-   a) The work must carry prominent notices stating that you modified
    it, and giving a relevant date.
-   b) The work must carry prominent notices stating that it is
    released under this License and any conditions added under
    section 7. This requirement modifies the requirement in section 4
    to "keep intact all notices".
-   c) You must license the entire work, as a whole, under this
    License to anyone who comes into possession of a copy. This
    License will therefore apply, along with any applicable section 7
    additional terms, to the whole of the work, and all its parts,
    regardless of how they are packaged. This License gives no
    permission to license the work in any other way, but it does not
    invalidate such permission if you have separately received it.
-   d) If the work has interactive user interfaces, each must display
    Appropriate Legal Notices; however, if the Program has interactive
    interfaces that do not display Appropriate Legal Notices, your
    work need not make them do so.

A compilation of a covered work with other separate and independent
works, which are not by their nature extensions of the covered work,
and which are not combined with it such as to form a larger program,
in or on a volume of a storage or distribution medium, is called an
"aggregate" if the compilation and its resulting copyright are not
used to limit the access or legal rights of the compilation's users
beyond what the individual works permit. Inclusion of a covered work
in an aggregate does not cause this License to apply to the other
parts of the aggregate.

#### 6. Conveying Non-Source Forms.

You may convey a covered work in object code form under the terms of
sections 4 and 5, provided that you also convey the machine-readable
Corresponding Source under the terms of this License, in one of these
ways:

-   a) Convey the object code in, or embodied in, a physical product
    (including a physical distribution medium), accompanied by the
    Corresponding Source fixed on a durable physical medium
    customarily used for software interchange.
-   b) Convey the object code in, or embodied in, a physical product
    (including a physical distribution medium), accompanied by a
    written offer, valid for at least three years and valid for as
    long as you offer spare parts or customer support for that product
    model, to give anyone who possesses the object code either (1) a
    copy of the Corresponding Source for all the software in the
    product that is covered by this License, on a durable physical
    medium customarily used for software interchange, for a price no
    more than your reasonable cost of physically performing this
    conveying of source, or (2) access to copy the Corresponding
    Source from a network server at no charge.
-   c) Convey individual copies of the object code with a copy of the
    written offer to provide the Corresponding Source. This
    alternative is allowed only occasionally and noncommercially, and
    only if you received the object code with such an offer, in accord
    with subsection 6b.
-   d) Convey the object code by offering access from a designated
    place (gratis or for a charge), and offer equivalent access to the
    Corresponding Source in the same way through the same place at no
    further charge. You need not require recipients to copy the
    Corresponding Source along with the object code. If the place to
    copy the object code is a network server, the Corresponding Source
    may be on a different server (operated by you or a third party)
    that supports equivalent copying facilities, provided you maintain
    clear directions next to the object code saying where to find the
    Corresponding Source. Regardless of what server hosts the
    Corresponding Source, you remain obligated to ensure that it is
    available for as long as needed to satisfy these requirements.
-   e) Convey the object code using peer-to-peer transmission,
    provided you inform other peers where the object code and
    Corresponding Source of the work are being offered to the general
    public at no charge under subsection 6d.

A separable portion of the object code, whose source code is excluded
from the Corresponding Source as a System Library, need not be
included in conveying the object code work.

A "User Product" is either (1) a "consumer product", which means any
tangible personal property which is normally used for personal,
family, or household purposes, or (2) anything designed or sold for
incorporation into a dwelling. In determining whether a product is a
consumer product, doubtful cases shall be resolved in favor of
coverage. For a particular product received by a particular user,
"normally used" refers to a typical or common use of that class of
product, regardless of the status of the particular user or of the way
in which the particular user actually uses, or expects or is expected
to use, the product. A product is a consumer product regardless of
whether the product has substantial commercial, industrial or
non-consumer uses, unless such uses represent the only significant
mode of use of the product.

"Installation Information" for a User Product means any methods,
procedures, authorization keys, or other information required to
install and execute modified versions of a covered work in that User
Product from a modified version of its Corresponding Source. The
information must suffice to ensure that the continued functioning of
the modified object code is in no case prevented or interfered with
solely because modification has been made.

If you convey an object code work under this section in, or with, or
specifically for use in, a User Product, and the conveying occurs as
part of a transaction in which the right of possession and use of the
User Product is transferred to the recipient in perpetuity or for a
fixed term (regardless of how the transaction is characterized), the
Corresponding Source conveyed under this section must be accompanied
by the Installation Information. But this requirement does not apply
if neither you nor any third party retains the ability to install
modified object code on the User Product (for example, the work has
been installed in ROM).

The requirement to provide Installation Information does not include a
requirement to continue to provide support service, warranty, or
updates for a work that has been modified or installed by the
recipient, or for the User Product in which it has been modified or
installed. Access to a network may be denied when the modification
itself materially and adversely affects the operation of the network
or violates the rules and protocols for communication across the
network.

Corresponding Source conveyed, and Installation Information provided,
in accord with this section must be in a format that is publicly
documented (and with an implementation available to the public in
source code form), and must require no special password or key for
unpacking, reading or copying.

#### 7. Additional Terms.

"Additional permissions" are terms that supplement the terms of this
License by making exceptions from one or more of its conditions.
Additional permissions that are applicable to the entire Program shall
be treated as though they were included in this License, to the extent
that they are valid under applicable law. If additional permissions
apply only to part of the Program, that part may be used separately
under those permissions, but the entire Program remains governed by
this License without regard to the additional permissions.

When you convey a copy of a covered work, you may at your option
remove any additional permissions from that copy, or from any part of
it. (Additional permissions may be written to require their own
removal in certain cases when you modify the work.) You may place
additional permissions on material, added by you to a covered work,
for which you have or can give appropriate copyright permission.

Notwithstanding any other provision of this License, for material you
add to a covered work, you may (if authorized by the copyright holders
of that material) supplement the terms of this License with terms:

-   a) Disclaiming warranty or limiting liability differently from the
    terms of sections 15 and 16 of this License; or
-   b) Requiring preservation of specified reasonable legal notices or
    author attributions in that material or in the Appropriate Legal
    Notices displayed by works containing it; or
-   c) Prohibiting misrepresentation of the origin of that material,
    or requiring that modified versions of such material be marked in
    reasonable ways as different from the original version; or
-   d) Limiting the use for publicity purposes of names of licensors
    or authors of the material; or
-   e) Declining to grant rights under trademark law for use of some
    trade names, trademarks, or service marks; or
-   f) Requiring indemnification of licensors and authors of that
    material by anyone who conveys the material (or modified versions
    of it) with contractual assumptions of liability to the recipient,
    for any liability that these contractual assumptions directly
    impose on those licensors and authors.

All other non-permissive additional terms are considered "further
restrictions" within the meaning of section 10. If the Program as you
received it, or any part of it, contains a notice stating that it is
governed by this License along with a term that is a further
restriction, you may remove that term. If a license document contains
a further restriction but permits relicensing or conveying under this
License, you may add to a covered work material governed by the terms
of that license document, provided that the further restriction does
not survive such relicensing or conveying.

If you add terms to a covered work in accord with this section, you
must place, in the relevant source files, a statement of the
additional terms that apply to those files, or a notice indicating
where to find the applicable terms.

Additional terms, permissive or non-permissive, may be stated in the
form of a separately written license, or stated as exceptions; the
above requirements apply either way.

#### 8. Termination.

You may not propagate or modify a covered work except as expressly
provided under this License. Any attempt otherwise to propagate or
modify it is void, and will automatically terminate your rights under
this License (including any patent licenses granted under the third
paragraph of section 11).

However, if you cease all violation of this License, then your license
from a particular copyright holder is reinstated (a) provisionally,
unless and until the copyright holder explicitly and finally
terminates your license, and (b) permanently, if the copyright holder
fails to notify you of the violation by some reasonable means prior to
60 days after the cessation.

Moreover, your license from a particular copyright holder is
reinstated permanently if the copyright holder notifies you of the
violation by some reasonable means, this is the first time you have
received notice of violation of this License (for any work) from that
copyright holder, and you cure the violation prior to 30 days after
your receipt of the notice.

Termination of your rights under this section does not terminate the
licenses of parties who have received copies or rights from you under
this License. If your rights have been terminated and not permanently
reinstated, you do not qualify to receive new licenses for the same
material under section 10.

#### 9. Acceptance Not Required for Having Copies.

You are not required to accept this License in order to receive or run
a copy of the Program. Ancillary propagation of a covered work
occurring solely as a consequence of using peer-to-peer transmission
to receive a copy likewise does not require acceptance. However,
nothing other than this License grants you permission to propagate or
modify any covered work. These actions infringe copyright if you do
not accept this License. Therefore, by modifying or propagating a
covered work, you indicate your acceptance of this License to do so.

#### 10. Automatic Licensing of Downstream Recipients.

Each time you convey a covered work, the recipient automatically
receives a license from the original licensors, to run, modify and
propagate that work, subject to this License. You are not responsible
for enforcing compliance by third parties with this License.

An "entity transaction" is a transaction transferring control of an
organization, or substantially all assets of one, or subdividing an
organization, or merging organizations. If propagation of a covered
work results from an entity transaction, each party to that
transaction who receives a copy of the work also receives whatever
licenses to the work the party's predecessor in interest had or could
give under the previous paragraph, plus a right to possession of the
Corresponding Source of the work from the predecessor in interest, if
the predecessor has it or can get it with reasonable efforts.

You may not impose any further restrictions on the exercise of the
rights granted or affirmed under this License. For example, you may
not impose a license fee, royalty, or other charge for exercise of
rights granted under this License, and you may not initiate litigation
(including a cross-claim or counterclaim in a lawsuit) alleging that
any patent claim is infringed by making, using, selling, offering for
sale, or importing the Program or any portion of it.

#### 11. Patents.

A "contributor" is a copyright holder who authorizes use under this
License of the Program or a work on which the Program is based. The
work thus licensed is called the contributor's "contributor version".

A contributor's "essential patent claims" are all patent claims owned
or controlled by the contributor, whether already acquired or
hereafter acquired, that would be infringed by some manner, permitted
by this License, of making, using, or selling its contributor version,
but do not include claims that would be infringed only as a
consequence of further modification of the contributor version. For
purposes of this definition, "control" includes the right to grant
patent sublicenses in a manner consistent with the requirements of
this License.

Each contributor grants you a non-exclusive, worldwide, royalty-free
patent license under the contributor's essential patent claims, to
make, use, sell, offer for sale, import and otherwise run, modify and
propagate the contents of its contributor version.

In the following three paragraphs, a "patent license" is any express
agreement or commitment, however denominated, not to enforce a patent
(such as an express permission to practice a patent or covenant not to
sue for patent infringement). To "grant" such a patent license to a
party means to make such an agreement or commitment not to enforce a
patent against the party.

If you convey a covered work, knowingly relying on a patent license,
and the Corresponding Source of the work is not available for anyone
to copy, free of charge and under the terms of this License, through a
publicly available network server or other readily accessible means,
then you must either (1) cause the Corresponding Source to be so
available, or (2) arrange to deprive yourself of the benefit of the
patent license for this particular work, or (3) arrange, in a manner
consistent with the requirements of this License, to extend the patent
license to downstream recipients. "Knowingly relying" means you have
actual knowledge that, but for the patent license, your conveying the
covered work in a country, or your recipient's use of the covered work
in a country, would infringe one or more identifiable patents in that
country that you have reason to believe are valid.

If, pursuant to or in connection with a single transaction or
arrangement, you convey, or propagate by procuring conveyance of, a
covered work, and grant a patent license to some of the parties
receiving the covered work authorizing them to use, propagate, modify
or convey a specific copy of the covered work, then the patent license
you grant is automatically extended to all recipients of the covered
work and works based on it.

A patent license is "discriminatory" if it does not include within the
scope of its coverage, prohibits the exercise of, or is conditioned on
the non-exercise of one or more of the rights that are specifically
granted under this License. You may not convey a covered work if you
are a party to an arrangement with a third party that is in the
business of distributing software, under which you make payment to the
third party based on the extent of your activity of conveying the
work, and under which the third party grants, to any of the parties
who would receive the covered work from you, a discriminatory patent
license (a) in connection with copies of the covered work conveyed by
you (or copies made from those copies), or (b) primarily for and in
connection with specific products or compilations that contain the
covered work, unless you entered into that arrangement, or that patent
license was granted, prior to 28 March 2007.

Nothing in this License shall be construed as excluding or limiting
any implied license or other defenses to infringement that may
otherwise be available to you under applicable patent law.

#### 12. No Surrender of Others' Freedom.

If conditions are imposed on you (whether by court order, agreement or
otherwise) that contradict the conditions of this License, they do not
excuse you from the conditions of this License. If you cannot convey a
covered work so as to satisfy simultaneously your obligations under
this License and any other pertinent obligations, then as a
consequence you may not convey it at all. For example, if you agree to
terms that obligate you to collect a royalty for further conveying
from those to whom you convey the Program, the only way you could
satisfy both those terms and this License would be to refrain entirely
from conveying the Program.

#### 13. Use with the GNU Affero General Public License.

Notwithstanding any other provision of this License, you have
permission to link or combine any covered work with a work licensed
under version 3 of the GNU Affero General Public License into a single
combined work, and to convey the resulting work. The terms of this
License will continue to apply to the part which is the covered work,
but the special requirements of the GNU Affero General Public License,
section 13, concerning interaction through a network will apply to the
combination as such.

#### 14. Revised Versions of this License.

The Free Software Foundation may publish revised and/or new versions
of the GNU General Public License from time to time. Such new versions
will be similar in spirit to the present version, but may differ in
detail to address new problems or concerns.

Each version is given a distinguishing version number. If the Program
specifies that a certain numbered version of the GNU General Public
License "or any later version" applies to it, you have the option of
following the terms and conditions either of that numbered version or
of any later version published by the Free Software Foundation. If the
Program does not specify a version number of the GNU General Public
License, you may choose any version ever published by the Free
Software Foundation.

If the Program specifies that a proxy can decide which future versions
of the GNU General Public License can be used, that proxy's public
statement of acceptance of a version permanently authorizes you to
choose that version for the Program.

Later license versions may give you additional or different
permissions. However, no additional obligations are imposed on any
author or copyright holder as a result of your choosing to follow a
later version.

#### 15. Disclaimer of Warranty.

THERE IS NO WARRANTY FOR THE PROGRAM, TO THE EXTENT PERMITTED BY
APPLICABLE LAW. EXCEPT WHEN OTHERWISE STATED IN WRITING THE COPYRIGHT
HOLDERS AND/OR OTHER PARTIES PROVIDE THE PROGRAM "AS IS" WITHOUT
WARRANTY OF ANY KIND, EITHER EXPRESSED OR IMPLIED, INCLUDING, BUT NOT
LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
A PARTICULAR PURPOSE. THE ENTIRE RISK AS TO THE QUALITY AND
PERFORMANCE OF THE PROGRAM IS WITH YOU. SHOULD THE PROGRAM PROVE
DEFECTIVE, YOU ASSUME THE COST OF ALL NECESSARY SERVICING, REPAIR OR
CORRECTION.

#### 16. Limitation of Liability.

IN NO EVENT UNLESS REQUIRED BY APPLICABLE LAW OR AGREED TO IN WRITING
WILL ANY COPYRIGHT HOLDER, OR ANY OTHER PARTY WHO MODIFIES AND/OR
CONVEYS THE PROGRAM AS PERMITTED ABOVE, BE LIABLE TO YOU FOR DAMAGES,
INCLUDING ANY GENERAL, SPECIAL, INCIDENTAL OR CONSEQUENTIAL DAMAGES
ARISING OUT OF THE USE OR INABILITY TO USE THE PROGRAM (INCLUDING BUT
NOT LIMITED TO LOSS OF DATA OR DATA BEING RENDERED INACCURATE OR
LOSSES SUSTAINED BY YOU OR THIRD PARTIES OR A FAILURE OF THE PROGRAM
TO OPERATE WITH ANY OTHER PROGRAMS), EVEN IF SUCH HOLDER OR OTHER
PARTY HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH DAMAGES.

#### 17. Interpretation of Sections 15 and 16.

If the disclaimer of warranty and limitation of liability provided
above cannot be given local legal effect according to their terms,
reviewing courts shall apply local law that most closely approximates
an absolute waiver of all civil liability in connection with the
Program, unless a warranty or assumption of liability accompanies a
copy of the Program in return for a fee.

END OF TERMS AND CONDITIONS

### How to Apply These Terms to Your New Programs

If you develop a new program, and you want it to be of the greatest
possible use to the public, the best way to achieve this is to make it
free software which everyone can redistribute and change under these
terms.

To do so, attach the following notices to the program. It is safest to
attach them to the start of each source file to most effectively state
the exclusion of warranty; and each file should have at least the
"copyright" line and a pointer to where the full notice is found.

        <one line to give the program's name and a brief idea of what it does.>
        Copyright (C) <year>  <name of author>

        This program is free software: you can redistribute it and/or modify
        it under the terms of the GNU General Public License as published by
        the Free Software Foundation, either version 3 of the License, or
        (at your option) any later version.

        This program is distributed in the hope that it will be useful,
        but WITHOUT ANY WARRANTY; without even the implied warranty of
        MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
        GNU General Public License for more details.

        You should have received a copy of the GNU General Public License
        along with this program.  If not, see <https://www.gnu.org/licenses/>.

Also add information on how to contact you by electronic and paper
mail.

If the program does terminal interaction, make it output a short
notice like this when it starts in an interactive mode:

        <program>  Copyright (C) <year>  <name of author>
        This program comes with ABSOLUTELY NO WARRANTY; for details type `show w'.
        This is free software, and you are welcome to redistribute it
        under certain conditions; type `show c' for details.

The hypothetical commands \`show w' and \`show c' should show the
appropriate parts of the General Public License. Of course, your
program's commands might be different; for a GUI interface, you would
use an "about box".

You should also get your employer (if you work as a programmer) or
school, if any, to sign a "copyright disclaimer" for the program, if
necessary. For more information on this, and how to apply and follow
the GNU GPL, see <https://www.gnu.org/licenses/>.

The GNU General Public License does not permit incorporating your
program into proprietary programs. If your program is a subroutine
library, you may consider it more useful to permit linking proprietary
applications with the library. If this is what you want to do, use the
GNU Lesser General Public License instead of this License. But first,
please read <https://www.gnu.org/licenses/why-not-lgpl.html>.
```
