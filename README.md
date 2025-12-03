# ollama-facade-inspection-costing

## 1\. Local AI Inference Engine Setup (PC & Ollama)

1.  **Model Pull:** Ensure vision model (e.g., a fine-tuned **LLaVA** or **Ollama's Llama-Vision** variant) is pulled and running on local machine.
2.  **Python Integration:** Use the official **`ollama` Python library** to interact with the local server from a Python script running on PC.
3.  **Prompt Engineering for Structured Output:** Crucially, prompt to the vision model should instruct it to output the results in a predictable, easy-to-parse **JSON format**. This allows the subsequent Python script to ingest the data directly.

**Example Prompting Goal:**
*Input:* Image of a facade.
*Instruction:* "Identify all instances of 'Cracks,' 'Spalling,' and 'Bulging Brick' (referencing NYC DOB glossary). For each defect, output a JSON object containing the `defect_type` and a list of pixel coordinates for the bounding box or segmentation mask."

-----

## 2\. Structured Defect Output

The Ollama model's output should be designed to feed directly into quantification script:

| Key | Description | Example Value |
| :--- | :--- | :--- |
| **`defect_id`** | Unique identifier for the instance. | `C101` |
| **`defect_type`** | The classification (from glossary). | `"Spalled Concrete"` |
| **`segmentation_mask`** | Polygon coordinates (in pixels). | `[[100, 250], [150, 250], [150, 300], ...]` |
| **`reference_points`** | (Optional but useful) Anchor points if measured by the model. | `(X_pixel_1, Y_pixel_1)` |

-----

## 3\. The Core Challenge: Pixel-to-Real-World Calibration 📏

This is the most critical step for **quantitative analysis**. Since images are 2D and subject to perspective distortion, we must introduce a calibration step to convert pixel counts (Area and Lineal Feet) into real-world units.

The simplest, most robust 2D method is using **Reference Objects** and **Homography**.

1.  **On-Site Calibration:** When taking facade photos, place a common object of **known physical dimension** (e.g., a 1-foot square checkerboard, a fixed-length pole, or a ruler) near the area of the defect.

2.  **Homography/Perspective Correction (OpenCV):** Use the Python library **OpenCV** to calculate the **Homography matrix**. This matrix uses the known dimensions of reference object (e.g., 12 inches) compared to its size in pixels (e.g., 200 pixels) to find the ratio for that specific plane in the image. This technique mathematically flattens the facade plane in the image, correcting for skew.

3.  **Scale Factor Calculation:** Once corrected, we calculate the scale:
    $$\text{Scale Factor (SF)} = \frac{\text{Real-World Length}}{\text{Pixel Length}}$$
    $$\text{Example: } SF = \frac{1 \text{ ft}}{200 \text{ pixels}}$$

-----

## 4\. Python Quantification Script (NumPy & Pandas)

Create a script that reads the JSON output from Ollama and processes it using the calculated scale factor. we can run this script either on local PC or upload the JSON data to **Google Colab** to leverage its cloud resources for batch processing and visualization.

### A. Area Measurement (Square Feet)

For defects requiring area (e.g., Spalling, Deteriorated Mortar):

1.  Use the `segmentation_mask` coordinates to create a binary mask image.
2.  Count the total number of white pixels in the mask (**A\_pixels**).
3.  Apply the scale factor (squared):

$$\text{Area (sq ft)} = \text{A\pixels} \times (SF)^2$$

### B. Lineal Measurement (Lineal Feet)

For defects requiring length (e.g., Cracks):

1.  Use algorithms like the **skeletonization** or **Douglas-Peucker algorithm** (available in libraries like `scikit-image` or `shapely`) on the segmentation mask to find the center line of the crack.
2.  Calculate the pixel length of that centerline (**L\_pixels**).
3.  Apply the scale factor:

$$\text{Length (lin ft)} = \text{L\pixels} \times SF$$

-----

## 5\. Cost Integration and Final Reporting

The final step is to merge quantified data with cost-per-unit metrics.

1.  **Create a Cost Database:** Maintain a simple Python dictionary or a CSV file (e.g., using **Pandas**) defining the cost per unit for each defect type:

    ```python
    COST_DATABASE = {
        "Spalled Concrete": {"unit": "sq ft", "cost_per_unit": 75.00},
        "Cracks": {"unit": "lin ft", "cost_per_unit": 35.00},
        "Bulging Brick": {"unit": "unit", "cost_per_unit": 1200.00} # for an anchor/pin
    }
    ```

2.  **Calculate Total Cost:** Iterate through quantified defects and multiply the measured quantity by the relevant unit cost.

3.  **Generate Report:** Use libraries like **Pandas** to structure the final data and **Matplotlib/Seaborn** to generate visualizations of the facade image overlaid with the quantified defects and a summary table for the final quantitative analysis report.
