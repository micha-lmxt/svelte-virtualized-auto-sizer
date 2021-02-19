# svelte-virtualized-auto-sizer

> Svelte port of the AutoSizer component from react-virtualized.

High-order component that automatically adjusts the width and height of a single child. Read more about this component in [my blog](https://gradientdescent.de/porting-react)

[![NPM](https://img.shields.io/npm/v/svelte-virtualized-auto-sizer.svg)](https://www.npmjs.com/package/svelte-virtualized-auto-sizer)

## Install

```bash
npm install --save-dev svelte-virtualized-auto-sizer
```

## Documentation

Please see API documentation of "AutoSizer" in `react-virtualized` package [here](https://github.com/bvaughn/react-virtualized/blob/master/docs/AutoSizer.md).

On the props style and children are removed, since these are React-specific.

Instead, use slot props:

```html

<AutoSizer let:width={childWidth} let:height={childHeight}>
    <div 
    style={"width:"+childWidth+"px;height:"+childHeight+"px;"}
    >Test</div>
</AutoSizer>

```

