<script>
    import { error } from "@sveltejs/kit";
    import { Button } from "@sveltestrap/sveltestrap";
    import { get } from "svelte/store";
    import { getAllContexts, onMount } from "svelte";
    import HintButton from "./HintButton.svelte";

    let letters = [];
    let definition = [];
    let inputValue = '';
    let statusText;
    let resetBtn;

    //change the definition
    let definitionCount = 0;
    let definitionExport;
    $: definitionExport = definition[definitionCount];

    let score = 0;

    onMount(() => {
        getWord();
    });

    function getWord() {
       //get random word
       return fetch(`https://random-word-api.vercel.app/api?words=1`)
            .then((response) => {
                if(!response.ok){
                    throw new Error('Network response was not ok. Status: ' + response.status);
                }
                return response.json();
            })
            .then((values) => {
                const randomWord = values[0];
                letters = randomWord.toUpperCase().split('');
                getDefinition();
                //delete this
                console.log(letters);
            })
            .catch  (error => {console.error('Error: ', error)});
    }

    function getDefinition(){
        //get definition of the word
        fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${letters.join('')}`)
            .then(response => {
                if(!response.ok){
                    getWord();
                    throw new Error('Network response was not ok Status: ' + response.status);
                }
                return response.json();
            })
            .then(data => {
                definition = data[0].meanings[0].definitions;
                for(let i = 0; i < definition.length; i++){
                    definition[i] = definition[i].definition;
                }
                definitionExport = definition[definitionCount];
            })
            .catch(error => {console.error('Error: ', error)});
    }

    function checkWord(e, index){
        if(statusText){
            return;
        }
        const input = e.target;
        if(input.value.toUpperCase() === letters[index]){
            input.style.backgroundColor = 'green';
            input.readOnly = true;
        }else if(input.value === undefined || input.value === null || input.value === ''){
            input.style.backgroundColor = 'transparent';
        } else {
            input.style.backgroundColor = 'red';
        }

        if (input.value === '') {
            input.focus();
            event.preventDefault();
        } else if (index >= letters.length - 1) {
            
        } else {
            const nextInput = document.querySelector(`input[data-index="${index + 1}"]`);
            nextInput.focus();
        }
        
        const inputs = document.querySelectorAll('input');
        const allReadOnly = Array.prototype.every.call(inputs, (input) => {
            return input.readOnly;
        });
        if(allReadOnly){
            statusText = `Correct! ${letters.join('')}`;
            setTimeout(() => resetBtn.focus(), 100);
        }
    }

    function inputPress(e, index){
        if (e?.key === 'Backspace') {
            const prevInput = document.querySelector(`input[data-index="${index - 1}"]`);
            if (prevInput && prevInput.readOnly === false) {
                prevInput.focus();
                e.target.value = '';
            }
        }
    }

    function resetForm(){
        document.getElementById('form').reset();
        statusText = '';
        score++;
        getWord().then(() => {
            const inputs = document.querySelectorAll('input');
            inputs.forEach((input, index) => {
                input.readOnly = false;
                if(index === 0){
                    input.focus();
                }
                input.style.backgroundColor = 'transparent';
            });
        });
    }

    function onHintClick(){
        const randomIndex = Math.floor(Math.random() * letters.length);
        const inputs = document.querySelectorAll('input');
        inputs[randomIndex].value = letters[randomIndex];
    }
</script>

<div>
    <h1 class="text-center mt-5">Guess the word</h1>
    <div>Your score: {score}</div>
    <HintButton {onHintClick}/>
    {#if statusText}
        <div>{statusText}</div>
        <button bind:this={resetBtn} class="btn btn-secondary" onclick={() => resetForm()}>Next word</button>
    {/if}
    <form id="form" class="d-flex justify-content-center align-items-center flex-column" style="height: 60vh;">
        <div class="justify-content-center align-items-center">
            {#if definitionCount > 0}
                <button class="btn btn-secondary" onclick={() => definitionCount--}>←</button>
            {/if}
                {definitionExport}
            {#if definitionCount < definition.length - 1}
                <button class="btn btn-secondary" onclick={() => definitionCount++}>→</button>
            {/if}
        </div>
        <div class="outputText mb-5">
            {#each letters as letter, index (letter + index)}
                <input type="text" onkeydown={(e) => inputPress(e, index)} data-index={index} oninput={(e) => checkWord(e, index)} maxlength="1"/>
            {/each}
        </div>
    </form>
</div>

<style>
    .outputText {
        font-size: 4rem;
    }

    .outputText input {
        user-select: all;
        width: 5rem;
        border: none;
        outline: none;
        text-align: center;
        margin-right: 20px;
        display: inline-block;
        text-transform: uppercase;
        background-color: transparent;
        border-bottom: solid 2px rgb(20, 181, 230);
    }
</style>