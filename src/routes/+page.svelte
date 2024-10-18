<script>
    import Header from './../Components/Header.svelte';
    import Button from '../Components/Button.svelte';
    import Footer from './../Components/Footer.svelte';
    import Map from "../Components/Map.svelte";
    import Label from "../Components/Label.svelte";
    import { ButtonType } from "../lib/styles";

    let inputValue;
    let selectedFeatureTypes = [];
    let showContent = false;

    function handleButtonClick(type) {
        if (selectedFeatureTypes.includes(type)) {
            selectedFeatureTypes = selectedFeatureTypes.filter(t => t !== type); // Odeber typ, pokud už je vybraný
        } else {
            selectedFeatureTypes.push(type); // Přidej typ, pokud není vybraný
        }
    }

    // Function to go back to the initial state (button only)
    function handleHeaderClick() {
        showContent = false;
    }
</script>

{#if !showContent}
    <div class="flex items-center justify-center min-h-screen bg-gray-100">
        <Button onClick={() => showContent = true} btStyle={ButtonType.CIRCLE}>
            JeMiZle
        </Button>
    </div>
{/if}

{#if showContent}
    <!-- The header title becomes clickable -->
    <Header title="Najdi lékaře" on:click={handleHeaderClick} />
    <div class="flex items-center justify-center bg-gray-100">
        <Button onClick={() => handleButtonClick('zubni')} btStyle={ButtonType.zubar}>
            zubař
        </Button>
        <Button onClick={() => handleButtonClick('prakticky')} btStyle={ButtonType.praktickylekar}>
            praktický lékař
        </Button>
        <Button onClick={() => handleButtonClick('detsky')} btStyle={ButtonType.gynekolog}>
            dětský lékař
        </Button>
        <Button onClick={() => handleButtonClick('gynekolog')} btStyle={ButtonType.gynekolog}>
            gynekologie
        </Button>
    </div>

    <div class="my-8">
        <Map {inputValue} {selectedFeatureTypes} />
    </div>

    <div class="flex justify-center mb-8">
        <Label bind:value={inputValue} />
    </div>

    <Footer />
{/if}
