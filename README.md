# ivy-material-list-size

This repo shows a size regression with Ivy found in https://github.com/angular/angular-cli/issues/16866.

```sh
git clone https://github.com/filipesilva/ivy-material-list-size
cd ivy-material-list-size
yarn
yarn repro
```

The first build will use Ivy and output to `dist/ivy`, and the second build will use VE and output to `dist/ve`.


## What's happening?

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

You can use the command below to produce debug versions of the production bundles.
These debug versions keep the original variable names are are formatted for readability.
```
NG_BUILD_DEBUG_OPTIMIZE=minify ng build --prod && NG_BUILD_DEBUG_OPTIMIZE=minify ng build --configuration production,ve
```

The `mat-list-ivy.js` and `mat-list-ve.js` files contain contain the code from `@angular/material/list` that ends up in the production builds, as extracted manually from the debug builds.

The Ivy file retained the following classes:
```
MatListBase
MatListItemBase
MatNavList
MatList
MatListAvatarCssMatStyler
MatListIconCssMatStyler
MatListItem
MatListModule
```

The VE file retained the following classes:
```
MatListBase
MatListItemBase
list_MatNavList
list_MatList
list_MatListItem
MatListModule
```

Notably, the VE file does not reference either `MatListAvatarCssMatStyler` or `MatListIconCssMatStyler`.
In Ivy these two are referenced in the `MatListItem` factory code:

```js
return MatListItem.\u0275fac = function(t) {
    return new (t || MatListItem)(\u0275\u0275directiveInject(ElementRef), \u0275\u0275directiveInject(ChangeDetectorRef), \u0275\u0275directiveInject(list_MatNavList, 8), \u0275\u0275directiveInject(list_MatList, 8));
}, MatListItem.\u0275cmp = \u0275\u0275defineComponent({
    type: MatListItem,
    selectors: [ [ "mat-list-item" ], [ "a", "mat-list-item", "" ], [ "button", "mat-list-item", "" ] ],
    contentQueries: function(rf, ctx, dirIndex) {
        var _t;
        1 & rf && (\u0275\u0275contentQuery(dirIndex, list_MatListAvatarCssMatStyler, !0),
        \u0275\u0275contentQuery(dirIndex, list_MatListIconCssMatStyler, !0), \u0275\u0275contentQuery(dirIndex, core_MatLine, !0)),
        2 & rf && (\u0275\u0275queryRefresh(_t = \u0275\u0275loadQuery()) && (ctx._avatar = _t.first),
        \u0275\u0275queryRefresh(_t = \u0275\u0275loadQuery()) && (ctx._icon = _t.first),
        \u0275\u0275queryRefresh(_t = \u0275\u0275loadQuery()) && (ctx._lines = _t));
    },
    hostAttrs: [ 1, "mat-list-item" ],
    hostVars: 4,
    hostBindings: function(rf, ctx) {
        2 & rf && \u0275\u0275classProp("mat-list-item-avatar", ctx._avatar || ctx._icon)("mat-list-item-with-avatar", ctx._avatar || ctx._icon);
    },
    inputs: {
        disableRipple: "disableRipple"
    },
    exportAs: [ "matListItem" ],
    features: [ \u0275\u0275InheritDefinitionFeature ],
    ngContentSelectors: list_c2,
    decls: 6,
    vars: 2,
    consts: [ [ 1, "mat-list-item-content" ], [ "mat-ripple", "", 1, "mat-list-item-ripple", 3, "matRippleTrigger", "matRippleDisabled" ], [ 1, "mat-list-text" ] ],
    template: function(rf, ctx) {
        1 & rf && (\u0275\u0275projectionDef(list_c1), \u0275\u0275elementStart(0, "div", 0),
        \u0275\u0275elementStart(1, "div", 1, void 0), \u0275\u0275elementEnd(), \u0275\u0275projection(2),
        \u0275\u0275elementStart(3, "div", 2), \u0275\u0275projection(4, 1), \u0275\u0275elementEnd(),
        \u0275\u0275projection(5, 2), \u0275\u0275elementEnd()), 2 & rf && (1, selectIndexInternal(getTView(), getLView(), getSelectedIndex() + 1, getCheckNoChangesMode()),
        \u0275\u0275property("matRippleTrigger", ctx._getHostElement())("matRippleDisabled", ctx._isRippleDisabled()));
    },
    directives: [ core_MatRipple ],
    encapsulation: 2,
    changeDetection: 0
}), MatListItem;
```

Additionally, Ivy contains factory code (the `\u0275u0275defineComponent` calls) for `MatNavList`, `MatList`, and `MatListItem`, but VE only contains the equivalent code (`createRendererType2` calls) for `MatNavList` and `MatListItem`. Since these factories contain a large amount of css code, the bundle size increase is noticeable.
