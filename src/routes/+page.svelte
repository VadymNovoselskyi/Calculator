<script>
    const OPERATORS_ORDER = ['/×', '+-']; // Represents order of operations (1st - '×' and '/', 2nd - '+' and '-')
    const decimalRoundUp = 1e6; // Used to round up answers to allow up to 6 decimal precision

    const operatorEndMatch = /[\+\-×\/]$/; // Matches one '+', '-', '×' or '/' at the end
    const digitEndMatch = /\d$/; // Matches one 0-9 at the end

    let display = '';

    function validateDidgit(event) {
        const digit = event.target.innerText;

        // Does not allow '0' to be at the beginning of a number (otherwise i.e 01 would be possible)
        const digitCommaEndMatch = /[\d\.]$/; // Matches 0-9 and '.'
        if(digit == 0 && !(digitCommaEndMatch.test(display))) return;

        display += digit;
    }

    function validateOperator(event) {
        const operator = event.target.innerText;
        
        // Check if display ends with a digit (0-9)
        if (digitEndMatch.test(display)) {
            display += operator;
        }
        // Check if 'operator' is '-' and display ends with another operator ('+', '-', '×' or '/') or is empty
        // Otherwise several '(-)' in a row could be possible
        else if (operator === '-' && (operatorEndMatch.test(display) || display === '')) {
            display += '(-)'; // Adds '-' as '(-)' to be treated as the sign for number that comes after it 
        }
    }

    function validatePoint() {
        // Validates that there is no preceding points that might break arithmetics
        const numberAfterPointMatch = /(?<=\.)\d+$/; // Matches one or more digits (0-9) at the end if the digit(s) is preceded by a point ('.')
        const pointEndMatch = /\.$/; // Matches point ('.') at the end
        const invalidDecimalState = numberAfterPointMatch.test(display) || pointEndMatch.test(display);
        if (invalidDecimalState) return; 
        
        // Check if display ends with an operator ('+', '-', '×' or '/') or display is empty
        if (operatorEndMatch.test(display) || display === '') {
            display += '0.'; //Adds '0' in front of the '.' in order to "create" a new number
        }
        else {
            display += '.';
        }
    }

    function deleteLast() {
        // If display is empty there is nothing to delete
        if (display === '') return;

        // If the last chars in display are '(-)' 3 chars should be deleted instead of 1
        const trailingNegative = /\(-\)$/.test(display); // Tests if '(-)' is at the end
        if (trailingNegative) {
            display = display.slice(0, display.length - 3);
        }
        else {
            display = display.slice(0, display.length - 1);
        }
    }

    function calculate() {
        // Erase everything unnecessary (non-digits) at the and of the display
        const allNonDigitEndMatch = /\D+$/ // Matches one or more non-digit ('(', ')', '+', '-', '×' or '/') at the end 
        const displayValue = display.replace(allNonDigitEndMatch, '');
        
        // Extract every number from display with regard to its sign ('+' or '-')
        const allOperatorsButMinusInBracketsMatch = /(?<!\(\-)[\+\-×\/](?!\))/g // Matches all operators ('+', '-', '×' or '/') except for '-' that are inside Brackets '(-)'
        const allMinusInBracketsMatch = /\(-\)/g; //Matches all '(-)'
        const numbers = displayValue.split(allOperatorsButMinusInBracketsMatch).map(number => number.replace(allMinusInBracketsMatch, '-'));

        // Extract every operator ('+', '-', '×' or '/') from display except for '-' that are inside brackets '(-)'
        const allOperatorsMatch = /[\+\-×\/]/g // Matches all operators ('+', '-', '×' or '/')
        const operators = (displayValue.replace(allMinusInBracketsMatch, '').match(allOperatorsMatch)); 

        // Continue if 'numbers' and 'operators' are not empty
        if(!numbers || !operators) {
            display = ''; // Clears the display
            return;
        }

        let answer = 0; // Stores the answer to user's input
        const calculateIndexes = []; // Stores indexes of calculated operations
        
        OPERATORS_ORDER.forEach(searchedOperator => { // Loops through operations in order
            operators.forEach((operator, index) => { // Loops through every operator exctracted from the display
                // Check if its 'operator's turn to be calculated
                if (!searchedOperator.includes(operator)) return;
                
                // Calculate number of 'deletedIndex' (indexes of operators that were alreary calculated) smaller than 'index'
                const indexToSubstract = calculateIndexes.reduce(
                    (sum, deletedIndex) => (sum += deletedIndex < index ? 1 : 0),
                    0
                );
                index -=  indexToSubstract;

                const number1 = Number(numbers[index]); // Gets number on the left side of 'operator'
                const number2 = Number(numbers[index + 1]); // Gets number on the right side of 'operator'
                const result = handleArithmetics(operator, number1, number2); // Calcualtes the result of 'number1' 'operator' 'number2'

                numbers[index] = result; // Changes number on the right of 'operator' to 'result'
                numbers.splice(index + 1, 1); // Removes number on the left of 'operator'
                
                // Add 'operator's index to the 'calculateIndexes' in order to keep track of indexes that were already calculated
                calculateIndexes.push(index);

                answer = result;
            });
        });
        display = `${parseInt(answer * decimalRoundUp) / decimalRoundUp}`; // Displays answer with a round up
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
        <button id="add" on:click={validateOperator}>+</button>

        <button on:click={validateDidgit}>4</button>
        <button on:click={validateDidgit}>5</button>
        <button on:click={validateDidgit}>6</button>
        <button id="sub" on:click={validateOperator}>-</button>

        <button on:click={validateDidgit}>1</button>
        <button on:click={validateDidgit}>2</button>
        <button on:click={validateDidgit}>3</button>
        <button id="mul" on:click={validateOperator}>×</button>
        
        <button id="point" on:click={validatePoint}>.</button>
        <button on:click={validateDidgit}>0</button>
        <button id="equal" on:click={calculate}>=</button>
        <button id="div" on:click={validateOperator}>/</button>

        <button id="delete" style = 'grid-column: span 2;'  on:click={deleteLast}>DELETE</button>
        <button id="clear" style = 'grid-column: span 2;'  on:click={() => display=''}>CLEAR</button>
    </section>
</main>