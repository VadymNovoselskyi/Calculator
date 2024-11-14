<script>
    const OPERATORS_ORDER = ['/×', '+-'];
    const decimalRoundUp = 1e6;

    const operatorEndCheck = /[\+\-×\/]$/; // Matches one '+', '-', '×' or '/' at the end
    const digitEndCheck = /\d$/; // Matches one 0-9 at the end

    let display = '';

    function validateDidgit(event) {
        const digit = event.target.innerText;

        // Does not allow '0' to be the beginning of some number (otherwise ex 01 would be possible)
        const digitCommaEndCheck = /[\d\.]$/; // Matchesatches 0-9 and '.'
        if(digit == 0 && !(digitCommaEndCheck.test(display))) return;

        // Add digit to the display
        display += digit;
    }

    function validateOperator(event) {
        const operator = event.target.innerText;
        
        // Check if display ends with an operator ('+', '-', '×' or '/') or display is empty
        if (operatorEndCheck.test(display) || display === '') {
            if(operator === '-') display += '(-)'; // Adds '-' as '(-)' to be treated as the sign for number that comes after it 
            return;
        }

        // Check if display ends with a digit (0-9)
        else if (digitEndCheck.test(display)) {
            // Add operator to the display
            display += operator;
        }
    }

    function validateComma() {
        // Check if display ends with an operator ('+', '-', '×' or '/') or display is empty
        if (operatorEndCheck.test(display) || display === '') {
            display += '0.'; //Adds '0' in front of the '.' in order to "create" a new number
            return;
        }

        // Validates that there is no preceding points that might break arithmetics
        const numberAfterPoint = /(?<=\.)\d+$/; // Matches one or more digits (0-9) at the end if the digit(s) is preceded by a point ('.')
        const pointEndCheck = /\.$/; // Matches point ('.') at the end
        if(numberAfterPoint.test(display) || pointEndCheck.test(display)) return; 

        // Add comma to the display
        display += '.';
    }

    function calculate() {
        // Erase everything unnecessary (non-digits) at the ond of display
        const allNonDigitEndCheck = /\D+$/ // Matches one or more non-digit ('(', ')', '+', '-', '×' or '/') at the end 
        const displayValue = display.replace(allNonDigitEndCheck, '');
        
        // Extract every number from display with regard to its sign ('+' or '-')
        const allOperatorsButMinusInParentheses = /(?<!\(\-)[\+\-×\/](?!\))/g // Matches all operators ('+', '-', '×' or '/') except for '-' that are inside parentheses '(-)'
        const allMinusInParentheses = /\(-\)/g; //Matches all '(-)'
        const numbers = displayValue.split(allOperatorsButMinusInParentheses).map(number => number.replace(allMinusInParentheses, '-'));

        // Extract every operator ('+', '-', '×' or '/') from display except for '-' that are inside parentheses '(-)'
        const allOperators = /[\+\-×\/]/g // Matches all operators ('+', '-', '×' or '/')
        const operators = (displayValue.replace(allMinusInParentheses, '').match(allOperators)); 

        // Continue if 'numbers' and 'operators' are not empty
        if(!numbers || !operators) {
            display = ''; // Clears the display
            return;
        }

        let answer = 0; // Stores the answer to user's input
        const calculateIndexes = []; // Stores indexes of calculated operations
        
        OPERATORS_ORDER.forEach(searchedOperator => { // Loops through operators in order they come (1st - '×' and '/', 2nd - '+' and '-')
            operators.forEach((operator, index) => { //Loops through every operator exctracted from the display
                // Check if its 'operator's turn to be calculated or if 'operator' should be calculated in the next iteration of 'OPERATORS_ORDER'
                if (!searchedOperator.includes(operator)) return;
                
                // Calculate number of 'deletedIndex' smaller than 'index' that were alreary calculated
                const indexToSubstract = calculateIndexes.reduce(
                    (sum, deletedIndex) => sum += deletedIndex < index ? 1 : 0
                    , 0
                );
                index -=  indexToSubstract;

                const number1 = Number(numbers[index]); // Gets number on the left side of 'operator'
                const number2 = Number(numbers[index + 1]); // Gets number on the right side of 'operator'
                const result = handleArithmetics(operator, number1, number2); // Calcualtes the result of 'number1' 'operator' 'number2'

                numbers[index] = result; // Changes number on the right of 'operator' to 'result'
                numbers.splice(index + 1, 1); // Removes number on the left of 'operator'
                
                // Adds'operator's index to the 'calculateIndexes' in order to keep track of indexes that were already calculated
                calculateIndexes.push(index);

                // Assign 'result' to 'answer'
                answer = result;
            });
        });
        display = ''; // Clears the display
        display = `${parseInt(answer * decimalRoundUp) / decimalRoundUp}`;
    }

    function handleArithmetics(operator, number1, number2) {
        // Perform the right calculation based on the operator
        switch (operator) {
            case '/': return number1 / number2;
            case '×': return number1 * number2;
            case '+': return number1 + number2;
            case '-': return number1 - number2;
        }
    }
</script>

<svelte:head>
    <title>Kalkylator</title>
</svelte:head>

<main>
    <input type="text" id = 'lcd' value={display} disabled>
    <section id = 'keyBoard'>
        <button on:click={validateDidgit}>7</button>
        <button on:click={validateDidgit}>8</button>
        <button on:click={validateDidgit}>9</button>
        <button on:click={validateOperator}>+</button>

        <button on:click={validateDidgit}>4</button>
        <button on:click={validateDidgit}>5</button>
        <button on:click={validateDidgit}>6</button>
        <button on:click={validateOperator}>-</button>

        <button on:click={validateDidgit}>1</button>
        <button on:click={validateDidgit}>2</button>
        <button on:click={validateDidgit}>3</button>
        <button on:click={validateOperator}>×</button>
        
        <button on:click={validateComma}>,</button>
        <button on:click={validateDidgit}>0</button>
        <button on:click={calculate}>=</button>
        <button on:click={validateOperator}>/</button>

        <button id='clear' style = 'grid-column: span 4;'  on:click={() => display=''}>CLEAR</button>
    </section>
</main>