**Question:**

Create a JavaScript function `calculateTax` that accurately determines income tax based on a series of tax slabs. Each slab is defined by a start value, end value, and a tax percentage. The function must adhere to the following requirements:

1. **Slab Processing:** Tax slabs may not be provided in ascending order. The function should first sort them by their start values to ensure correct calculation order.
2. **Range Handling:** Each slab's tax percentage applies only to the portion of the income that falls within its [start, end) range (i.e., inclusive of `start`, exclusive of `end`). If the remaining income exceeds the slab's range, tax the entire range. Otherwise, tax only the remaining income.
3. **Error Checking:** 
   - Validate that `income` is positive and slabs are valid non-empty arrays.
   - Ensure no slab has an invalid range (e.g., `start` ≥ `end`).
   - Throw an error if the slabs do not fully cover the income (e.g., remaining income after processing all slabs is still positive).

**Solution:**
function calculate_tax(income, tax_slabs) {
    if (income <= 0 || !Array.isArray(tax_slabs) || tax_slabs.length === 0) {
        throw new Error('Invalid income or tax slabs');
    }

    const sortedSlabs = tax_slabs.sort((a, b) => a.start - b.start);

    let tax = 0;
    let remainingIncome = income;

    for (const slab of sortedSlabs) {
        const { start, end, p: percentage } = slab;

        if (remainingIncome <= 0) break;

        if (start >= end) throw new Error('Invalid slab range');

        const slabRange = end - start;

        const taxableAmount = Math.min(remainingIncome, slabRange);

        tax += (taxableAmount * percentage) / 100;

        remainingIncome -= taxableAmount;
    }

    if (remainingIncome > 0) {
        throw new Error('Insufficient slabs to cover income');
    }

    return tax;
}
// Test case 1
try {
    console.log(
        calculate_tax(150000, [
            { start: 0, end: 100000, p: 0 },
            { start: 100001, end: 500000, p: 2 },
        ])
    );

    // Test case 2
    console.log(
        calculate_tax(400000, [
            { start: 0, end: 100000, p: 0 },
            { start: 100001, end: 300000, p: 1 },
            { start: 300001, end: 500000, p: 2 },
        ])
    );

    // Test case 3
    console.log(
        calculate_tax(600000, [
            { start: 0, end: 300000, p: 0 },
            { start: 300001, end: 500000, p: 1 },
        ])
    );

} catch (err) {
    console.error(err.message);
}
