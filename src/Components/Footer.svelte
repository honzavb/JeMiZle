<!-- src/components/Footer.svelte -->
<slot>
    <footer class="bg-gray-800 text-white py-4">
    <div class="container mx-auto text-center">
        <p>&copy; {new Date().getFullYear()}</p>
    </div>
</footer>
</slot>