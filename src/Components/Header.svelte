<script>
    export let title = "";
</script>

<slot>
    <header class="bg-blue-600 text-white py-4 shadow">
        <div class="container mx-auto px-4">
            <h1 class="text-2xl font-bold text-center cursor-pointer" on:click>
                {title}
            </h1>
        </div>
    </header>
</slot>

