<script>
    let displayValue = '+';
    let operators = [];
    let values = [];

    function addDidgit(event) {
        displayValue += event.target.innerText;
    }

    function handleOperator(event) {
        let operator = event.target.innerText
        operators.push(operator);

        // const digitRegex = /.*[+\-*/](\w+)$/;
        // const match = displayValue.match(digitRegex);
        // console.log(match)

        displayValue += ` ${operator} `;
    }

    function addComma(event) {

    }

    function handleClick(event) {

    }

    function clear() {
        displayValue = 0;
        operators = []
        values = []
    }
</script>

<svelte:head>
    <title>Kalkylator</title>
</svelte:head>

<main>
    <input type="text" id = 'lcd' value={displayValue}>
    <section id = 'keyBoard'>
        <button id='b7' on:click={addDidgit}>7</button>
        <button id='b8' on:click={addDidgit}>8</button>
        <button id='b9' on:click={addDidgit}>9</button>
        <button id='add' on:click={handleOperator}>+</button>

        <button id='b4' on:click={addDidgit}>4</button>
        <button id='b5' on:click={addDidgit}>5</button>
        <button id='b6' on:click={addDidgit}>6</button>
        <button id='sub' on:click={handleOperator}>-</button>

        <button id='b1' on:click={addDidgit}>1</button>
        <button id='b2' on:click={addDidgit}>2</button>
        <button id='b3' on:click={addDidgit}>3</button>
        <button id='mul' on:click={handleOperator}>x</button>
        
        <button id='comma' on:click={addComma}>,</button>
        <button id='b0' on:click={addDidgit}>0</button>
        <button id='enter' on:click={handleClick}>=</button>
        <button id='div' on:click={handleOperator}>/</button>

        <button id='clear' style = 'grid-column: span 4;'  on:click={clear}>CLEAR</button>
    </section>
</main>