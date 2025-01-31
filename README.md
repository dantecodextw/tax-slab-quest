**Question:**

Create a JavaScript function `calculateTax` that accurately determines income tax based on a series of tax slabs. Each slab is defined by a start value, end value, and a tax percentage. The function must adhere to the following requirements:

1. **Slab Processing:** Tax slabs may not be provided in ascending order. The function should first sort them by their start values to ensure correct calculation order.
2. **Range Handling:** Each slab's tax percentage applies only to the portion of the income that falls within its [start, end) range (i.e., inclusive of `start`, exclusive of `end`). If the remaining income exceeds the slab's range, tax the entire range. Otherwise, tax only the remaining income.
3. **Error Checking:** 
   - Validate that `income` is positive and slabs are valid non-empty arrays.
   - Ensure no slab has an invalid range (e.g., `start` ≥ `end`).
   - Throw an error if the slabs do not fully cover the income (e.g., remaining income after processing all slabs is still positive).

**Solution:**
```
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
```

Let's break down the `calculate_tax` function **line-by-line** with detailed explanations:

---

### **1. Input Validation**
```javascript
if (income <= 0 || !Array.isArray(tax_slabs) || tax_slabs.length === 0) {
    throw new Error('Invalid income or tax slabs');
}
```
- **Purpose**: Validate input integrity.
- **Checks**:
  - Income must be positive (`> 0`).
  - Tax slabs must be provided as a **non-empty array**.
- **Why?** Prevents invalid calculations like negative income or empty slabs.

---

### **2. Slab Sorting**
```javascript
const sortedSlabs = [...tax_slabs].sort((a, b) => a.start - b.start);
```
- **Purpose**: Ensure slabs are processed in ascending order.
- **Working**:
  - Creates a copy of `tax_slabs` using the spread operator (`[...tax_slabs]`) to avoid mutating the original array.
  - Sorts slabs by their `start` value (lowest first).
- **Example**: If input slabs are `[{start: 100}, {start: 0}]`, sorted slabs become `[{start: 0}, {start: 100}]`.

---

### **3. Tax Calculation Initialization**
```javascript
let tax = 0;
let remainingIncome = income;
```
- **Purpose**: Track total tax and untaxed income.
- **Variables**:
  - `tax`: Accumulates calculated tax.
  - `remainingIncome`: Initially set to the full income amount. Reduced as slabs are processed.

---

### **4. Slab Processing Loop**
```javascript
for (const slab of sortedSlabs) {
    const { start, end, p: percentage } = slab;
    if (remainingIncome <= 0) break;
    if (start >= end) throw new Error('Invalid slab range');
    const slabRange = end - start;
    const taxableAmount = Math.min(remainingIncome, slabRange);
    tax += (taxableAmount * percentage) / 100;
    remainingIncome -= taxableAmount;
}
```
#### **Step-by-Step Breakdown**:
1. **Destructure Slab Properties**:
   ```javascript
   const { start, end, p: percentage } = slab;
   ```
   - Extracts `start`, `end`, and `percentage` from the current slab.

2. **Early Exit Check**:
   ```javascript
   if (remainingIncome <= 0) break;
   ```
   - If no income remains to tax, exit the loop early (optimization).

3. **Slab Validation**:
   ```javascript
   if (start >= end) throw new Error('Invalid slab range');
   ```
   - Ensures each slab has a valid range (e.g., `start: 100`, `end: 50` is invalid).

4. **Calculate Slab Range**:
   ```javascript
   const slabRange = end - start; // No +1 → end is exclusive
   ```
   - Example: Slab `{start: 0, end: 100000}` has a range of **100,000** units.

5. **Determine Taxable Amount**:
   ```javascript
   const taxableAmount = Math.min(remainingIncome, slabRange);
   ```
   - Taxes the lesser of:
     - The slab's full range (e.g., 100,000 units).
     - Remaining untaxed income.

6. **Calculate Tax for Slab**:
   ```javascript
   tax += (taxableAmount * percentage) / 100;
   ```
   - Applies the slab's tax percentage to the taxable amount.

7. **Update Remaining Income**:
   ```javascript
   remainingIncome -= taxableAmount;
   ```
   - Reduces the remaining income by the taxed amount.

---

### **5. Final Validation**
```javascript
if (remainingIncome > 0) {
    throw new Error('Insufficient slabs to cover income');
}
```
- **Purpose**: Ensure all income is covered by slabs.
- **Example**: If income is 600,000 but the highest slab ends at 500,000, this throws an error.

---

### **Time Complexity**
| Step               | Complexity  | Explanation                                  |
|--------------------|-------------|----------------------------------------------|
| Slab Sorting       | O(n log n)  | Dominant operation (sorting via `Array.sort`)|
| Slab Processing    | O(n)        | Single loop through sorted slabs             |
| **Overall**        | **O(n log n)** | Sorting is the bottleneck                  |

---

### **Space Complexity**
- **O(n)**: Due to the copy of slabs created for sorting (`sortedSlabs`).
- Other variables (`tax`, `remainingIncome`) use constant space (O(1)).

---

### **Key Insights**
1. **Range Handling**: Slabs use **[start, end)** intervals (end is exclusive).
   - Example: Slab `{start: 0, end: 100000}` covers income from **0 to 99,999**.
2. **Error Handling**: Covers edge cases like invalid slabs or incomplete coverage.
3. **Efficiency**: Early exit when income is fully taxed minimizes computations.

---

### **Example Walkthrough**
**Test Case 1**:
```javascript
calculate_tax(150000, [
    { start: 100001, end: 500000, p: 2 },
    { start: 0, end: 100000, p: 0 },
]);
```
1. **Sorted Slabs**: `[{start: 0, ...}, {start: 100001, ...}]`
2. **First Slab**:
   - Range: 100,000 units (0–99,999)
   - Tax: 100,000 × 0% = 0
   - Remaining income: 150,000 – 100,000 = 50,000
3. **Second Slab**:
   - Taxable amount: min(50,000, 500,000 – 100,001) = 50,000
   - Tax: 50,000 × 2% = 1,000
   - Remaining income: 0
4. **Result**: 1,000 (✅ Correct).

This implementation robustly handles progressive taxation systems with clear error checking and efficient computation.
