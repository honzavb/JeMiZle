<script>
    export let btStyle = '';
    export let value = '';    
    export let onClick;
</script>

<button on:click={onClick} class={btStyle}>
    <slot>{value}</slot>
</button>