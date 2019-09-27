<script context="module">
  import { integers } from "./ArrayHelpers.svelte";

  let defaults = {
    defaultItemHeight: 50,
    layerHeightOverViewHeight: 1 / 10,
    percentToFlip: 0.5,
    verticalSlack: 1, // To stagger items, use a number > 1.
    alignTop: true // false ==> align bottom
  };

  /**
   * The `repeater` function uses the provided `config` to customize
   * 3 functions which can be used to place an array of identically-
   * sized SVGs so that they span the full width of a view.
   *
   * Usage is straightforward. Specify:
   * - the size of the items being laid out
   * - the size of the view that they should span
   * - whether to align across the top or bottom of the view
   * - how tall the items should be as a proportion of the view's
   *   height (useful for reacting to view resizes)
   * - (optionally) whether to randomly stagger items vertically (as
   *   demonstrated by Billow's clouds).
   *
   * See `defaults` below for an example config.
   */
  export let xyt = (viewWidth, viewHeight, config) => {
    // 📝👆 It seems that reactivity doesn't work when all that's
    // changing is the _values_ in an object, so viewWidth and
    // can't be in the config.

    // 👇Nota bene: { a:1, b:2, b:3 } == { a:1, b:3 }
    const c = { ...defaults, ...config };

    const itemWidthOverItemHeight = c.defaultItemWidth / c.defaultItemHeight;
    const layerHeight = viewHeight * c.layerHeightOverViewHeight;
    const itemHeight = layerHeight / c.verticalSlack;
    const itemWidth = itemHeight * itemWidthOverItemHeight;
    const itemScale = itemHeight / c.defaultItemHeight;
    const itemOverlap = itemWidth ? 0.1 * itemWidth : 0;
    const itemCount = Math.ceil(viewWidth / (itemWidth - 2 * itemOverlap));

    // Returns a horizontal flip transform `perc` percent of the time,
    // where `perc` is a value between 0 and 1.
    //
    // This is the simplest way I could think to keep the items from
    // looking too repetetive.
    const flipPercentageOfClouds = perc => {
      const flipHorizontal = cw => {
        return `scale(-1,1) translate(-${cw},0)`;
      };
      return Math.random() > 0.2 ? flipHorizontal(itemWidth) : "";
    };

    const transform = () => {
      return `${flipPercentageOfClouds(c.percentToFlip)} scale(${itemScale})`;
    };

    const count = () => Math.ceil(viewWidth / (itemWidth - 2 * itemOverlap));

    const y = () => {
      return c.alignTop
        ? (layerHeight - itemHeight) * Math.random()
        : viewHeight - itemHeight;
    };

    return {
      x: idx => {
        return idx * (itemWidth - 2 * itemOverlap) - itemOverlap;
      },

      y: y,

      transform: transform,
      count: count
    };
  };
</script>
