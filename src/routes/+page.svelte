<script>
    import { Button } from "@sveltestrap/sveltestrap";

    const alfabet = "abcdefghijklmnopqrstuvwxyz";
    const alfabetArray = alfabet.split("");

    let word = "Word";

    function getWord() {
        let randomLetter = alfabetArray[Math.floor(Math.random() * alfabetArray.length)];
        fetch(`https://api.datamuse.com/words?sp=${randomLetter}*`)
            .then((response) => response.json())
            .then((values) => {
                const randomWord = values[Math.floor(Math.random() * values.length)].word;
                word = randomWord;
            })
            .catch((error) => console.error(error));
    }

</script>

<div class="d-flex justify-content-center align-items-center flex-column" style="height: 70vh;">
    <div id="outputText">{word.toUpperCase()}</div>
    <input type="text" id="inputText" /><br />
    <button on:click={getWord}>Submit</button>
</div>

<style>
    #outputText {
        font-size: 10rem;
    }
</style>
