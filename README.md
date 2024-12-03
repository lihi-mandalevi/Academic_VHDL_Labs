# Image Filtering Project


---

## Project Structure
The repository is organized as follows:

```
├── README.md              # Main project documentation (this file)
├── Lab1_ImageFilters/     # Contains lab-specific resources
│   ├── utilities/         # Contains Python utility scripts and related resources
│   │   ├── raw_image_to_mif.py    # Converts RAW image files into MIF files for Quartus
│   │   ├── filters.py             # Applies and compares image filters on RAW images
│   │   ├── README_HE.md           # Detailed explanation in Hebrew about the filtering algorithms
│   ├── VHDL/              # Folder for VHDL source code and MIF files
│   │   ├── *.vhd          # VHDL files for implementation
│   │   ├── *.mif          # MIF files generated for Quartus
```

---

## Files in `utilities/` Folder

### 1. `raw_image_to_mif.py`
- **Description**:
  This script reads a RAW image file (e.g., `256x256` with RGB channels), processes the pixel data, and generates three separate MIF files for the **R**, **G**, and **B** channels. The MIF files can be used to initialize ROM blocks in Quartus for hardware simulations.

- **How to Use**:
  1. Place your RAW image file (e.g., `lena_005noise_256x256.raw`) in the same directory as the script.
  2. Update the RAW file path in the script if necessary.
  3. Run the script:
     ```bash
     python raw_image_to_mif.py
     ```
  4. The generated MIF files (`r_channel.mif`, `g_channel.mif`, and `b_channel.mif`) will be saved in the `utilities/` folder.
  5. Copy the MIF files to the appropriate location (e.g., `Lab1_ImageFilters/VHDL/`) for Quartus usage.

---

### 2. `filters.py`
- **Description**:
  This script applies three types of filters (**Median**, **Median of Medians**, and **Conditional Median of Medians**) on a RAW image. It reads the RAW file, processes each RGB channel individually, and displays the original and filtered images side-by-side for comparison.

- **How to Use**:
  1. Place your RAW image file (e.g., `lena_005noise_256x256.raw`) in the same directory as the script.
  2. Update the RAW file path in the `main` function if necessary.
  3. Run the script:
     ```bash
     python filters.py
     ```
  4. The script will display the original image and the results of each filter, allowing for easy visual comparison.

---

## Hebrew Explanation
For a detailed explanation in Hebrew about the different filtering algorithms and their uses, refer to the **README_HE.md** file in the `utilities/` folder. This document provides an in-depth overview of each filter type, including **Median**, **Median of Medians**, and **Conditional Median of Medians**.

---

## Notes
- The `filters.py` script includes visual output to help you evaluate the effectiveness of each filter on the image.
- Ensure you have the necessary Python packages installed before running the scripts (e.g., `numpy`, `scipy`, `PIL`, `matplotlib`).

To install the required packages, you can use:
```bash
pip install numpy scipy pillow matplotlib
```

---

## Getting Started with VS Code
To get started with this project using **VS Code**, follow these steps. Please follow each step carefully and type the exact commands into your terminal.

1. **Clone the Repository or Download the Files to Your Computer**:
   - If you choose to clone the repository, open your terminal (command line) and type the following commands. If you downloaded the files, you can skip to the next step.
   ```bash
   git clone <repository_url>
   cd <repository_directory>
   ```
   - This command will download the entire project to your computer and move into the project's main directory.

2. **Create a Virtual Environment**:
   - A virtual environment is a separate Python setup for this project. This keeps all installed packages isolated for just this project.
   - In VS Code, right-click on one of the files, and select 'Open in Integrated Terminal'. This will open a terminal inside VS Code.
   - Then type the following commands in the terminal:
   ```bash
   python -m venv venv
   ```
   - This command will create a folder named `venv` that contains the virtual environment.

3. **Activate the Virtual Environment**:
   - You need to activate the virtual environment to use it. Follow the instructions based on your operating system:
   - **Windows**: In your terminal, type:
     ```bash
     venv\Scripts\activate
     ```
     - After typing this command, you should see `(venv)` at the beginning of the command line, indicating the virtual environment is active.
   - **macOS/Linux**: In your terminal, type:
     ```bash
     source venv/bin/activate
     ```
     - You should also see `(venv)` at the beginning of your command line.

4. **Install Required Packages**:
   - We need to install some packages to help run our scripts, like handling images and numerical calculations.
   - With the virtual environment activated, type the following command:
   ```bash
   pip install numpy scipy pillow matplotlib
   ```
   - This command will download and install all the necessary libraries for the project. Make sure you see a successful installation message for each package.

5. **Run the Scripts**:
   - You have two options to run the scripts:
     1. **Option 1: Using the Terminal**
        - Make sure your terminal is open within VS Code and type:
        ```bash
        python filters.py
        ```
        - This will execute the script and display the results on your screen.
     2. **Option 2: Using the Run Button**
        - Alternatively, you can click the **Run** button located in the top right corner of VS Code to run the script directly.
   - Follow similar steps for running other scripts like `raw_image_to_mif.py`.





---

# FPGA Image Processing Project

## Overview

This project aims to implement an image processing system using an FPGA, specifically the **ALTERA DE2** board, to perform **Median of Medians** filtering on an image. The system processes the image by handling each of the primary color channels (Red, Green, Blue) separately and applying a 3x3 sliding window filter for noise removal (specifically salt-and-pepper noise) in each color channel.

---

## System Architecture

The FPGA system is designed to process an image by handling the Red (R), Green (G), and Blue (B) color channels separately in parallel. The image is represented in 8-bit color depth (one byte per pixel), and the median filtering operation is performed on a 3x3 pixel window for each color.

### Key Components of the System

1. **ROM (Read-Only Memory)**  
   - The ROM stores the input image data for each color channel (R, G, B).  
   - Each ROM unit contains **256 addresses**, with each address holding **2048 bits** of data representing 256 pixels. Each pixel is represented by 8 bits (color depth of 8 bits).
   - There are **3 ROM units**, one for each color (R, G, B).
   - **Output size of ROM for each color**: **2048 bits**.

2. **Registers**  
   - The Registers connect the ROM to the Buffers, holding the data of one row from the ROM.
   - **Input size of each Register**: **2048 bits** (for 256 pixels per row).
   - The Register reads data from one address in the ROM and transfers it to the corresponding Buffer.
   - **Purpose of Registers**: The Registers serve as intermediate storage to synchronize the data flow between the ROM and Buffers, ensuring proper timing and avoiding glitches.

3. **Buffers**  
   - Buffers temporarily store data for processing. Each color channel has **3 Buffers** (one for each row in the 3x3 window: Top, Middle, Bottom).
   - **Input size of each Buffer**: **2048 bits** (for 256 pixels per row).
   - Initially, 3 reads are required to fill the Buffers. After that, the Buffers hold the data of 3 consecutive rows. As new data is read from the Registers, the data in the Buffers shifts up, and the new row of pixels is added to the bottom row of the Buffer for processing.
   - The Buffers are used for processing the 3x3 sliding window mask that computes the Median of Medians for each pixel.

4. **RAM (Random Access Memory)**  
   - The processed image is stored in RAM after filtering is completed. 
   - **Output size of RAM for each color**: **2048 bits** per row, totaling **2048 bits x 256 pixels** for each color.
   - **Purpose of RAM**: RAM stores the final filtered output image data for each color channel (R, G, B).

5. **Median of Medians Filter**  
   - The **Median of Medians filter** is applied using a 3x3 sliding window, which calculates the median of 9 neighboring pixels for each pixel. The filter operates separately on each color channel (R, G, B).
   - **Median Calculation**: For each 3x3 window, the filter selects the median value of the 9 pixels and applies it to the corresponding pixel in the output.

---

## System Design and Flow

### Memory Architecture

The system processes each color channel (R, G, B) independently, ensuring parallel processing for better performance. The steps of the image processing pipeline for one color (R, G, or B) are as follows:

1. **Input ROM**:  
   The ROM stores the raw image data for the selected color channel (R, G, or B).  
   - The ROM has **256 addresses** with each address holding **2048 bits** for 256 pixels.
  
2. **Registers**:  
   The Registers read the data for one row (2048 bits) from the ROM and pass it to the corresponding Buffer.  
   - Each Register is responsible for reading one row of data from the ROM (2048 bits) and transferring it to the Buffer.

3. **Buffers**:  
   The Buffers hold the current 3 rows (Top, Middle, Bottom) of image data for processing. The 3x3 sliding window is applied to each pixel in the window.  
   - The Buffers hold **2048 bits per row**, for a total of **2048 bits x 3 rows**.  
   - Initially, 3 read operations fill the Buffers with the data from the Registers. Subsequent read operations shift the data within the Buffers, allowing continuous processing of new data.

4. **Median Filter**:  
   The 3x3 sliding window calculates the median of the 9 pixels in the window and replaces the center pixel with this value.
   - The filtering operation is done independently for each color channel (R, G, B).

5. **Output Registers and RAM**:  
   After filtering is complete, the filtered image data for each color channel is written to the output Registers and then stored in the RAM.
   - Each color channel (R, G, B) is written to a separate RAM module, and each RAM stores **2048 bits per row** of processed data.

---

### System Block Diagram

```
+---------------------+
|    Line Buffer 1    |  <---+  
+---------------------+       |    
            ^                 |
            |                 |
+---------------------+       |
|    Input ROM        |       |
|   R      G      B   |       |
+---------------------+       |
            |                 |
+---------------------+       |
|    Input Registers  |       |
+---------------------+       |
            |                 |
+---------------------+       |
|    Input Buffer     |       |
+---------------------+       |
            |                 |
+---------------------+       |
| Median Filter (3x3) |       |
|  Registers for Masks|       |
+---------------------+       |
            |                 |
+---------------------+       |
| Output Registers (RGB)|     |
+---------------------+       |
            |                 |
+---------------------+       |
|      Output RAM     |       |
+---------------------+       |
```

---

### Explanation of Buffers

The system includes 3 **Line Buffers** for each color channel (R, G, B), which store the current 3 rows of image data that are processed using the 3x3 sliding window. The Buffers are essential for handling the pixel values efficiently during the median filtering process.

- Each **Buffer** holds **2048 bits per row** (for 256 pixels) for each of the three rows: Top, Middle, and Bottom.
- The Buffers shift data as new rows are read, and each 3x3 window is processed by the filter on the Top, Middle, and Bottom rows.

In the first iteration, 3 reads fill the Buffers with data from the Registers. In subsequent iterations, the Buffers shift the rows upward, and the newest row is read from the Registers into the bottom row of the Buffers.

---

## Clocking and Synchronization

The FPGA operates with a clock speed of **50 MHz**. Proper synchronization between the different components (ROM, Registers, Buffers, and RAM) is crucial to ensure that data is transferred and processed without glitches.

- **Registers** are used between the ROM and Buffers to synchronize the data flow, ensuring that data is read and processed correctly in each clock cycle.
- A **Counter** is employed to track the row addresses and synchronize the flow of data between the components.

---

## FPGA Specifications

This project is designed to be implemented on the **ALTERA DE2** FPGA board, which offers:
- **LE (Logic Elements)** for implementing logic gates, flip-flops, and registers.
- **LAB (Logic Array Blocks)** to implement complex logic functions.
- **GI (Global Interconnect)** and **LI (Local Interconnect)** to connect the various components efficiently.

### Limitations of FPGA Resources
- The FPGA has a limited number of **FFs (Flip-Flops)** and **logic gates**, which can limit the number of Registers, Buffers, and other components that can be implemented. Efficient design is required to optimize resource usage and fit the design within the FPGA constraints.
  
---

## Conclusion

This FPGA-based image processing system demonstrates the use of parallel processing for each color channel (R, G, B), allowing efficient median filtering of a noisy image. The design employs Registers, Buffers, and a Median of Medians filter to process the image data and store the results in RAM. The use of the ALTERA DE2 FPGA enables high-speed processing and efficient memory management.

--- 

### **To Do:**
- Implement the VHDL code for ROM, Registers, Buffers, and the Median filter.
- Test the design on the ALTERA DE2 FPGA board.

