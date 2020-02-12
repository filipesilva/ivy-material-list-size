# ivy-material-list-size

This repo shows a size regression with Ivy.

```sh
git clone https://github.com/filipesilva/ivy-material-list-size
cd ivy-material-list-size
yarn
yarn repro
```

The first build will use Ivy and output to `dist/ivy`, and the second build will use VE and output to `dist/ve`.


## How is this change reproduced?

Given the following template:

```html
<mat-nav-list>
  <mat-list-item>
     hello
  </mat-list-item>
</mat-nav-list>
```

Ivy produces a 232 kB output while VE produces a 221 kB output, resulting in a 11 kB increase.

But if you change the template to:

```html
<mat-nav-list>
  hello
</mat-nav-list>
```

Ivy produces a 194 kB output while VE produces a 212 kB output, resulting in a 18 kB decrease.
