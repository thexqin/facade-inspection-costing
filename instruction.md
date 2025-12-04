# categorize only

> "Analyze the provided image of a building facade condition. Your task is to **identify the specific damage term/category** shown in the photo. Your final answer must explicitly state both the **Category** and the specific **Condition**, separated by a colon (e.g., 'Category: Condition').
>
> **Your answer must be chosen directly and verbatim from the list below.** Do not use any other terminology. If the damage is not explicitly listed, choose the closest applicable term or state 'Condition not explicitly listed'."

# quantification

> "Analyze the provided image of a building facade condition.
>
> **Step 1: Identification.** Identify the specific damage term/category shown in the photo. Your output for this step must be chosen directly and verbatim from the 'Facade Conditions List' provided below. Output as: `Identification: [Category]: [Condition]`.
>
> **Step 2: Quantification (Best Guess).** Based on the visual evidence in the photo, provide a best-guess estimate for the extent of the damage. Use a relevant measurement unit for the condition (e.g., number of bricks, lineal feet, square feet, count of units). If the image context is insufficient for a precise guess, state a **range** or the **countable items** visible. Output as: `Quantification: [Your Best Guess/Range] [Unit]`.
>
> **Step 3: Cost Unit Assignment.** Assign the most appropriate cost unit (e.g., 'Per Brick', 'Per Lineal Foot', 'Per Square Foot', 'Per Unit', 'Per Item') that would be used for repair estimation of the identified condition. Output as: `Cost Unit: [Assigned Cost Unit]`.
>
> **Final Answer Format:** Present your final answer using the following structure:
>
> **Identification:** [Category]: [Condition]\n
> **Quantification:** [Best Guess/Range] [Unit]
> **Cost Unit:** [Assigned Cost Unit]"
