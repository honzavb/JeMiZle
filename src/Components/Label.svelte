<script>
    export let value = 0;
  
    function handleInput(event) {
        value = +event.target.value; // Převod na číslo
    }
</script>
  
<label class="mt-4 flex flex-col items-center text-lg font-semibold text-gray-700">
    Radius (v km):
    <input 
        type="range" min="0" max="650" step="1" 
        bind:value={value} 
        on:input={handleInput}
        class="mt-2 w-32 p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-red-500 focus:border-transparent outline-none"
    />
    <span>{value / 10}</span>km
</label>
