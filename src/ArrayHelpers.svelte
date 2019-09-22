<script context="module">
  export const pickOne = arr => arr[~~(Math.random() * arr.length)];

  // https://stackoverflow.com/a/10050831/230615
  export const integers = (lowEnd, highEnd) => {
    return [...Array(highEnd - lowEnd + 1).keys()].map(i => i + lowEnd);
  };
</script>
