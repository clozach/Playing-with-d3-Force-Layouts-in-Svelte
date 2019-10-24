<script>
  // I'm particularly experienced with ARIA  standards, so there are
  // probably important details that will need to be tweaked here. For
  // now, I'm at least making it trivial to set a keyboad shortcut
  // for each button, and for that shortcut to be visibly reflected in
  // the button's visual presentation.
  import KeyHint from "./KeyHint.svelte";
  import { keypress } from "keypress.js";

  export let label;
  export let keyboardTrigger;
  export let action;

  let previousTrigger = keyboardTrigger;

  const listener = new keypress.Listener();
  $: {
    listener.unregister_combo(previousTrigger);
    listener.simple_combo(keyboardTrigger, action);
    previousTrigger = action;
  }
</script>

<button on:click={action}>
  {label}
  <KeyHint label="[{keyboardTrigger.toUpperCase()}]" />
</button>
