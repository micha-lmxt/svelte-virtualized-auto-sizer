<script>
    /// Ported from react-virtualized-auto-sizer
    /// MIT © bvaughn
    import { onDestroy, onMount } from "svelte";
    import { createDetectElementResize } from "./DetectElementResize";

    export let onResize = () => {};
    export let disableHeight = false;
    export let disableWidth = false;
    export let className = "";
    export let defaultHeight = undefined;
    export let defaultWidth = undefined;
    export let nonce = undefined;

    let height = defaultHeight || 0,
        width = defaultWidth || 0;
    let _parentNode, _autoSizer, _detectElementResize;

    const _onResize = () => {
        if (_parentNode) {
            // Guard against AutoSizer component being removed from the DOM immediately after being added.
            // This can result in invalid style values which can result in NaN values if we don't handle them.
            // See issue #150 for more context.

            const height_ = _parentNode.offsetHeight || 0;
            const width_ = _parentNode.offsetWidth || 0;

            const style = window.getComputedStyle(_parentNode) || {};
            const paddingLeft = parseInt(style.paddingLeft, 10) || 0;
            const paddingRight = parseInt(style.paddingRight, 10) || 0;
            const paddingTop = parseInt(style.paddingTop, 10) || 0;
            const paddingBottom = parseInt(style.paddingBottom, 10) || 0;

            const newHeight = height_ - paddingTop - paddingBottom;
            const newWidth = width_ - paddingLeft - paddingRight;

            if (
                (!disableHeight && height !== newHeight) ||
                (!disableWidth && width !== newWidth)
            ) {
                height = height_ - paddingTop - paddingBottom;
                width = width_ - paddingLeft - paddingRight;

                onResize({ height, width });
            }
        }
    };
    onMount(() => {
        if (
            _autoSizer &&
            _autoSizer.parentNode &&
            _autoSizer.parentNode.ownerDocument &&
            _autoSizer.parentNode.ownerDocument.defaultView &&
            _autoSizer.parentNode instanceof
                _autoSizer.parentNode.ownerDocument.defaultView.HTMLElement
        ) {
            // Delay access of parentNode until mount.
            // This handles edge-cases where the component has already been unmounted before its ref has been set,
            // As well as libraries like react-lite which have a slightly different lifecycle.
            _parentNode = _autoSizer.parentNode;

            // Defer requiring resize handler in order to support server-side rendering.
            // See issue #41
            _detectElementResize = createDetectElementResize(nonce);
            _detectElementResize.addResizeListener(_parentNode, _onResize);

            _onResize();
        } 
    });
    onDestroy(() => {
        if (_detectElementResize && _parentNode) {
            _detectElementResize.removeResizeListener(_parentNode, _onResize);
        }
    });
    const childParams = {};

    // Avoid rendering children before the initial measurements have been collected.
    // At best this would just be wasting cycles.
    let bailoutOnChildren = false,  outerstylewidth = false, outerstyleheight = false;

    $: {
        bailoutOnChildren = false;
        if (!disableHeight) {
            if (height === 0) {
                bailoutOnChildren = true;
            }
            outerstyleheight = true;
            childParams.height = height;
        }

        if (!disableWidth) {
            if (width === 0) {
                bailoutOnChildren = true;
            }
            outerstylewidth = true;
            childParams.width = width;
        }
    }
</script>

<style>
    div {
        overflow: visible;
    }
    .outerstylewidth {
        width:0;
    }
    .outerstyleheight{
        height:0;
    }
</style>

<div class={className} class:outerstylewidth class:outerstyleheight bind:this={_autoSizer}>
    {#if !bailoutOnChildren}
        <slot width={childParams.width} height={childParams.height} />
    {/if}
</div>
