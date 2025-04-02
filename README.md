# DGM4CAD
---

## README: Survey on Deep Generative Models for CAD

### Overview

This survey provides a comprehensive review of deep generative models applied to computer-aided design (CAD), highlighting their transformative potential in automating and enhancing the design process. We explore core paradigms such as Variational Autoencoders (VAEs), Generative Adversarial Networks (GANs), diffusion models, transformers, and GPT-based models, analyzing their applications in tasks like sketch generation, 3D shape synthesis, and text-to-CAD generation. Through an in-depth analysis of datasets, methods, and evaluation metrics, we identify the strengths and limitations of these models, ranging from high-quality output generation to challenges in training stability and computational efficiency. Additionally, we conduct a comparative study of large language model (LLM)-driven Python code generation for CAD tasks, evaluating models like Grok and DeepSeek-r1 for their creativity, accuracy, and reliability. As these technologies mature, they hold the promise of bridging human creativity and computational precision, ultimately redefining the landscape of design and engineering.

---

### Figure 1: Paradigms of Deep Generative Models for CAD

![Figure 1](./figs/figure_1_overview.png)

**Caption**: We summarize the paradigms centered on data modalities and task orientation. Input-driven tasks include text, sketch, and image-to-CAD, while CAD models or 3D shapes serve as inputs for tasks such as reconstruction and editing. Output modalities primarily focus on sketches, CAD models, and various 3D shapes, with a particular emphasis on sketch and CAD generation tasks. These tasks follow a typical pipeline, from requirement analysis to final output, illustrating how generative models guide the design process.

---

### Table 1: Datasets for CAD Generative Modeling

Below is a summary of key datasets used in CAD generative tasks, covering applications like sketch generation, CAD reconstruction, and text-to-CAD.

| **Application**       | **Input**          | **Output**            | **Data**     | **Method**                |
|-----------------------|--------------------|-----------------------|--------------|---------------------------|
| Sketch Generation     | Sketch             | Sketch                | 15,000,000+  | SketchGraphs [1]          |
|                       |                    |                       | 4,700,000+   | CAD as Language [2]       |
| CAD Reconstruction    | B-rep              | Construction sequence | 1,000,000+   | ABC [3]                   |
|                       |                    |                       | 37,000+      | CC3D-Ops [4]              |
|                       |                    |                       | 40,000+      | CADParser [5]             |
|                       |                    |                       | 8,625        | Fusion360 [6]             |
|                       | Construction sequence |                    | 179,133      | DeepCAD [7]               |
| Image-to-CAD          | Multi-view Images  |                       | 453,220      | Omni-CAD [8]              |
|                       | Single Image       |                       | 200,000+     | OpenECAD [9]              |
|                       |                    |                       | 4,574        | Img2CAD [10]              |
|                       |                    |                       | 208,853      | ABC-mono [11]             |
| Sketch-to-CAD         | Sketch             |                       | 82,000+      | Free2CAD [12]             |
| Text-to-CAD           | Text               |                       | 453,220      | Omni-CAD [8]              |
|                       |                    |                       | 158,000+     | Text2CAD [13]             |
|                       | Text               | Python code           | 200          | CADPrompt [14]            |
|                       |                    |                       | 57           | Query2CAD [15]            |

---

### Table 2: Generative Models for CAD Tasks

This table provides an overview of various generative models applied to CAD, categorized by their input, output, and model architecture.

| **Application**       | **Input**          | **Output**            | **Model**            | **Method**                |
|-----------------------|--------------------|-----------------------|----------------------|---------------------------|
| Sketch Generation     | None               | Sketch                | Transformer          | CurveGen/TurtleGen [16]   |
|                       | Sketch             |                       | Transformer          | CAD as Language [2]       |
|                       |                    |                       | VQ-VAE + Transformer | HNC-CAD [17]              |
| Sketch-to-CAD         | Sketch             | 3D CAD model          | GPT-4V + Diffusion   | Sketch2Prototype [18]     |
|                       |                    | 3D mesh               | Transformer          | PolyGen [19]              |
| CAD Generation        | None               | 3D CAD model          | Transformer          | CAD-MLLM [8]              |
|                       |                    | 3D shape              | GAN                  | GRASS [20]                |
| Text-to-CAD           | Text               | 3D CAD model          | Diffusion            | DreamGaussian [21]        |
|                       |                    | Python code           | GPT-4                | CADCodeVerify [14]        |
| Image-to-CAD          | Image              | 3D CAD model          | Autoencoder          | Deep CAD/CAE [22]         |
| Reconstruction        | 3D CAD model       | 3D CAD model          | VAE                  | SDM-NET [23]              |
| CAD Editing           | Partial shape      | 3D CAD model          | VQ-VAE + Transformer | HNC-CAD [17]              |

---

### Case Studies: Comparative Analysis of LLM-Driven CAD Generation

![Case Studies Comparison](figs/figure_3_case_studies.png)

**Caption**: Comparison of CAD models generated by different LLMs (o3-mini-high, DeepSeek-r1, Grok) based on Python code for various shapes. The figure illustrates the diverse levels of accuracy, creativity, and model quality exhibited by the evaluated models.

---

### Example Python Code Using Grok

Below is an example of Python code generated using Grok for a simple CAD task (e.g., generating a 3D model of a table). This code can be executed in FreeCAD to produce the corresponding CAD model.

```python
import FreeCAD
import Part

def create_table_model(tabletop_length, tabletop_width, tabletop_thickness, leg_height, leg_size, support_height, support_thickness):
    """
    Generate a 3D CAD model of a table in FreeCAD.
    
    Parameters:
    - tabletop_length (float): Length of the tabletop (X-direction)
    - tabletop_width (float): Width of the tabletop (Y-direction)
    - tabletop_thickness (float): Thickness of the tabletop (Z-direction)
    - leg_height (float): Height of the legs (Z-direction)
    - leg_size (float): Cross-sectional size of the legs (assumed square)
    - support_height (float): Vertical height of the structural supports
    - support_thickness (float): Thickness of the structural supports
    """
    
    # Create the tabletop
    # Positioned centered in XY, with base at Z=leg_height
    tabletop = Part.makeBox(
        tabletop_length,
        tabletop_width,
        tabletop_thickness,
        FreeCAD.Vector(-tabletop_length/2, -tabletop_width/2, leg_height),
        FreeCAD.Vector(0, 0, 1)
    )
    
    # Create the four legs
    # Legs extend from Z=0 to Z=leg_height, positioned at the corners
    leg1 = Part.makeBox(
        leg_size,
        leg_size,
        leg_height,
        FreeCAD.Vector(-tabletop_length/2, -tabletop_width/2, 0),  # Front-left
        FreeCAD.Vector(0, 0, 1)
    )
    leg2 = Part.makeBox(
        leg_size,
        leg_size,
        leg_height,
        FreeCAD.Vector(tabletop_length/2 - leg_size, -tabletop_width/2, 0),  # Front-right
        FreeCAD.Vector(0, 0, 1)
    )
    leg3 = Part.makeBox(
        leg_size,
        leg_size,
        leg_height,
        FreeCAD.Vector(-tabletop_length/2, tabletop_width/2 - leg_size, 0),  # Back-left
        FreeCAD.Vector(0, 0, 1)
    )
    leg4 = Part.makeBox(
        leg_size,
        leg_size,
        leg_height,
        FreeCAD.Vector(tabletop_length/2 - leg_size, tabletop_width/2 - leg_size, 0),  # Back-right
        FreeCAD.Vector(0, 0, 1)
    )
    
    # Create structural supports (aprons)
    # Positioned just below the tabletop, between the legs
    front_apron = Part.makeBox(
        tabletop_length - 2 * leg_size,
        support_thickness,
        support_height,
        FreeCAD.Vector(-tabletop_length/2 + leg_size, -tabletop_width/2, leg_height - support_height),
        FreeCAD.Vector(0, 0, 1)
    )
    back_apron = Part.makeBox(
        tabletop_length - 2 * leg_size,
        support_thickness,
        support_height,
        FreeCAD.Vector(-tabletop_length/2 + leg_size, tabletop_width/2 - support_thickness, leg_height - support_height),
        FreeCAD.Vector(0, 0, 1)
    )
    left_apron = Part.makeBox(
        support_thickness,
        tabletop_width - 2 * leg_size,
        support_height,
        FreeCAD.Vector(-tabletop_length/2, -tabletop_width/2 + leg_size, leg_height - support_height),
        FreeCAD.Vector(0, 0, 1)
    )
    right_apron = Part.makeBox(
        support_thickness,
        tabletop_width - 2 * leg_size,
        support_height,
        FreeCAD.Vector(tabletop_length/2 - support_thickness, -tabletop_width/2 + leg_size, leg_height - support_height),
        FreeCAD.Vector(0, 0, 1)
    )
    
    # Combine all parts into a single compound shape
    all_parts = [tabletop, leg1, leg2, leg3, leg4, front_apron, back_apron, left_apron, right_apron]
    table = Part.makeCompound(all_parts)
    
    # Add the model to the FreeCAD document
    doc = FreeCAD.activeDocument()
    if not doc:
        doc = FreeCAD.newDocument("Table")
    obj = doc.addObject("Part::Feature", "Table")
    obj.Shape = table
    doc.recompute()
    return obj

# Example usage
if __name__ == "__main__":
    create_table_model(
        tabletop_length=120.0,    # Length of the tabletop in mm
        tabletop_width=60.0,      # Width of the tabletop in mm
        tabletop_thickness=5.0,   # Thickness of the tabletop in mm
        leg_height=70.0,          # Height of the legs in mm
        leg_size=5.0,             # Cross-sectional size of the legs in mm
        support_height=10.0,      # Height of the structural supports in mm
        support_thickness=2.0     # Thickness of the structural supports in mm
    )
```

**Explanation**: This code fulfills the requirement to create a 3D CAD model of a table with the specified components and parameters using Python in FreeCAD.
