# CAD Code Generation Example

## Overview
Grok is an AI-powered assistant capable of generating code for various applications, including CAD design. This example demonstrates how Grok can produce functional Python code to create a 3D model in a CAD environment.

### gear
- [Prompt](#prompt): Provide the Python code to generate a simple gear shape in a CAD environment, including parameters for radius, teeth size, and central hole.
```python
import FreeCAD
import Part
import math

def create_gear(radius, num_teeth, tooth_height, hole_radius, thickness):
    """
    Generate a simple gear shape in FreeCAD with specified parameters.
    
    Parameters:
    - radius (float): The pitch radius of the gear (base circle radius in this simple model).
    - num_teeth (int): The number of teeth on the gear.
    - tooth_height (float): The height of each tooth (radial distance from root to tip is 2 * tooth_height).
    - hole_radius (float): The radius of the central hole.
    - thickness (float): The thickness (height) of the gear along the Z-axis.
    
    Returns:
    - gear_obj: The FreeCAD object representing the gear.
    """
    
    # Calculate angular parameters (in degrees)
    tooth_width_angle = 180.0 / num_teeth  # Angular width of each tooth
    delta = tooth_width_angle / 2          # Half the tooth width angle
    angle_step = 360.0 / num_teeth         # Angular step between teeth

    # Create a list of edges to define the 2D gear profile
    edges = []
    for i in range(num_teeth):
        # Base angle for the current tooth
        theta = i * angle_step
        
        # Define the four points of the rectangular tooth in the XY plane (z=0)
        p1 = FreeCAD.Vector(
            (radius - tooth_height) * math.cos(math.radians(theta - delta)),
            (radius - tooth_height) * math.sin(math.radians(theta - delta)),
            0
        )
        p2 = FreeCAD.Vector(
            (radius + tooth_height) * math.cos(math.radians(theta - delta)),
            (radius + tooth_height) * math.sin(math.radians(theta - delta)),
            0
        )
        p3 = FreeCAD.Vector(
            (radius + tooth_height) * math.cos(math.radians(theta + delta)),
            (radius + tooth_height) * math.sin(math.radians(theta + delta)),
            0
        )
        p4 = FreeCAD.Vector(
            (radius - tooth_height) * math.cos(math.radians(theta + delta)),
            (radius - tooth_height) * math.sin(math.radians(theta + delta)),
            0
        )
        
        # Add straight line edges for the tooth (inner to outer, across top, outer to inner)
        edges.append(Part.makeLine(p1, p2))
        edges.append(Part.makeLine(p2, p3))
        edges.append(Part.makeLine(p3, p4))
        
        # Define the starting point of the next tooth
        next_theta = ((i + 1) % num_teeth) * angle_step
        p1_next = FreeCAD.Vector(
            (radius - tooth_height) * math.cos(math.radians(next_theta - delta)),
            (radius - tooth_height) * math.sin(math.radians(next_theta - delta)),
            0
        )
        
        # Calculate the midpoint for the arc along the base circle
        mid_angle = (theta + delta + next_theta - delta) / 2
        mid_point = FreeCAD.Vector(
            (radius - tooth_height) * math.cos(math.radians(mid_angle)),
            (radius - tooth_height) * math.sin(math.radians(mid_angle)),
            0
        )
        
        # Add an arc along the base circle from p4 to p1_next
        arc = Part.Arc(p4, mid_point, p1_next)
        edges.append(arc.toShape())

    # Create a closed wire from the list of edges
    gear_wire = Part.Wire(edges)
    
    # Create a face from the wire
    gear_face = Part.Face(gear_wire)
    
    # Extrude the face to create the 3D gear
    gear_3d = gear_face.extrude(FreeCAD.Vector(0, 0, thickness))
    
    # Create a cylindrical hole through the gear
    hole = Part.makeCylinder(hole_radius, thickness)
    
    # Subtract the hole from the gear to create the final shape
    final_gear = gear_3d.cut(hole)
    
    # Add the gear to the FreeCAD document
    doc = FreeCAD.ActiveDocument
    if not doc:
        doc = FreeCAD.newDocument("Gear")
    gear_obj = doc.addObject("Part::Feature", "Gear")
    gear_obj.Shape = final_gear
    doc.recompute()
    
    return gear_obj

# Example usage
if __name__ == "__main__":
    gear = create_gear(
        radius=20.0,      # Pitch radius of 20 mm
        num_teeth=12,     # 12 teeth
        tooth_height=5.0, # Teeth extend 5 mm above and below the pitch radius
        hole_radius=5.0,  # Central hole radius of 5 mm
        thickness=10.0    # Gear thickness of 10 mm
    )
    print("Gear created successfully.")
'''

### bracket
- [Prompt](#prompt): Provide Python code to generate a simple bracket in a CAD system, with specified dimensions for length, width, and hole positions.
```python
import FreeCAD
import Part

def create_bracket(length, width, thickness, hole_diameter, hole_positions):
    """
    Create a simple bracket with specified dimensions and hole positions in FreeCAD.
    
    Parameters:
    - length: float, length of the bracket along the x-axis (in mm)
    - width: float, width of the bracket along the y-axis (in mm)
    - thickness: float, thickness of the bracket along the z-axis (in mm)
    - hole_diameter: float, diameter of the holes (in mm)
    - hole_positions: list of tuples, each tuple (x, y) specifies the center of a hole (in mm)
    """
    # Get the active document or create a new one if none exists
    doc = FreeCAD.ActiveDocument
    if not doc:
        doc = FreeCAD.newDocument("Bracket")
    
    # Create the base bracket shape
    bracket = Part.makeBox(length, width, thickness)
    
    # Create and cut each hole
    for pos in hole_positions:
        x, y = pos
        # Create a cylinder for the hole, positioned at (x, y, 0)
        hole = Part.makeCylinder(hole_diameter / 2, thickness, FreeCAD.Vector(x, y, 0), FreeCAD.Vector(0, 0, 1))
        # Cut the hole from the bracket
        bracket = bracket.cut(hole)
    
    # Create a Part feature in the document
    bracket_obj = doc.addObject("Part::Feature", "Bracket")
    bracket_obj.Shape = bracket
    
    # Recompute the document to update the view
    doc.recompute()
    
    return bracket_obj

# Example usage
bracket = create_bracket(
    length=100,
    width=50,
    thickness=5,
    hole_diameter=10,
    hole_positions=[(20, 25), (80, 25)]
)

### shaft with two holes at the ends
- [Prompt](#prompt): Generate Python code to model a shaft with two holes at the ends, including parameters for the shaft length, diameter, and hole size.
```python
import FreeCAD
import Part

def create_shaft(shaft_length, shaft_diameter, hole_size):
    """
    Create a shaft with two radial holes at the ends in FreeCAD.
    
    Parameters:
    - shaft_length (float): Length of the shaft along the Z-axis (in mm)
    - shaft_diameter (float): Diameter of the shaft (in mm)
    - hole_size (float): Diameter of the holes (in mm)
    
    Returns:
    - FreeCAD object: The resulting shaft object added to the document
    """
    # Get or create a FreeCAD document
    doc = FreeCAD.activeDocument()
    if not doc:
        doc = FreeCAD.newDocument("Shaft")
    
    # Calculate radii and hole offset
    shaft_radius = shaft_diameter / 2
    hole_radius = hole_size / 2
    hole_offset = shaft_diameter  # Distance from each end to the hole center
    
    # Create the shaft cylinder along the Z-axis
    shaft = Part.makeCylinder(shaft_radius, shaft_length, FreeCAD.Vector(0, 0, 0), FreeCAD.Vector(0, 0, 1))
    
    # Define the length of the hole cylinders (long enough to cut through the shaft)
    hole_length = 2 * shaft_diameter
    
    # Create the first hole cylinder along the X-axis at Z=hole_offset
    hole1 = Part.makeCylinder(hole_radius, hole_length, FreeCAD.Vector(-shaft_diameter, 0, hole_offset), FreeCAD.Vector(1, 0, 0))
    
    # Create the second hole cylinder along the X-axis at Z=shaft_length - hole_offset
    hole2 = Part.makeCylinder(hole_radius, hole_length, FreeCAD.Vector(-shaft_diameter, 0, shaft_length - hole_offset), FreeCAD.Vector(1, 0, 0))
    
    # Subtract the holes from the shaft
    shaft_with_hole1 = shaft.cut(hole1)
    shaft_with_holes = shaft_with_hole1.cut(hole2)
    
    # Add the resulting shape to the document
    shaft_obj = doc.addObject("Part::Feature", "Shaft")
    shaft_obj.Shape = shaft_with_holes
    doc.recompute()
    
    return shaft_obj

# Example usage
if __name__ == "__main__":
    create_shaft(100, 20, 5)

### circular part with multiple holes
- [Prompt](#prompt): Generate Python code for modeling a circular part with multiple holes, including parameters for diameter, number of holes, and hole size.
```python
import FreeCAD
import Part
import math

def create_holey_disk(disk_diameter, num_holes, hole_size, disk_thickness):
    """
    Create a circular disk with multiple holes in FreeCAD.
    
    Parameters:
    - disk_diameter (float): Diameter of the circular disk (in mm).
    - num_holes (int): Number of holes to be placed in the disk.
    - hole_size (float): Diameter of each hole (in mm).
    - disk_thickness (float): Thickness of the disk (in mm).
    
    Returns:
    - FreeCAD object: The resulting disk object with holes added to the document.
    """
    # Calculate radii
    disk_radius = disk_diameter / 2
    hole_radius = hole_size / 2
    
    # Set the radius for hole positions (75% of the disk radius)
    hole_position_radius = 0.75 * disk_radius
    
    # Create the main disk cylinder along the Z-axis
    disk = Part.makeCylinder(disk_radius, disk_thickness, FreeCAD.Vector(0, 0, 0), FreeCAD.Vector(0, 0, 1))
    
    # Create and subtract each hole
    for i in range(num_holes):
        # Calculate the angle for this hole in radians
        angle = 2 * math.pi * i / num_holes
        # Calculate the position of the hole
        x = hole_position_radius * math.cos(angle)
        y = hole_position_radius * math.sin(angle)
        # Create the hole cylinder
        hole = Part.makeCylinder(hole_radius, disk_thickness, FreeCAD.Vector(x, y, 0), FreeCAD.Vector(0, 0, 1))
        # Subtract the hole from the disk
        disk = disk.cut(hole)
    
    # Get the active document or create a new one if none exists
    doc = FreeCAD.activeDocument()
    if not doc:
        doc = FreeCAD.newDocument("HoleyDisk")
    
    # Add the resulting shape to the document
    obj = doc.addObject("Part::Feature", "HoleyDisk")
    obj.Shape = disk
    doc.recompute()
    
    return obj

# Example usage (uncomment to run in FreeCAD's Python console):
# create_holey_disk(disk_diameter=100, num_holes=6, hole_size=10, disk_thickness=10)

### rectangular part with three holes
- [Prompt](#prompt): Provide Python code to create a rectangular part with three holes in a CAD system, including parameters for dimensions and hole positions.
```python
import FreeCAD
import Part

def create_rectangular_part_with_holes(length, width, thickness, hole_diameters, hole_positions):
    """
    Create a rectangular part with three holes in FreeCAD.
    
    Parameters:
    - length (float): Length of the rectangle along the X-axis (in mm).
    - width (float): Width of the rectangle along the Y-axis (in mm).
    - thickness (float): Thickness of the rectangle along the Z-axis (in mm).
    - hole_diameters (list of float): Diameters of the three holes (in mm).
    - hole_positions (list of tuples): Positions of the three holes as (x, y) coordinates (in mm).
    
    Returns:
    - FreeCAD object: The resulting part object added to the document.
    """
    # Validate input parameters
    if length <= 0 or width <= 0 or thickness <= 0:
        raise ValueError("Dimensions must be positive.")
    if len(hole_diameters) != 3 or len(hole_positions) != 3:
        raise ValueError("Exactly three holes must be specified.")
    for d in hole_diameters:
        if d <= 0:
            raise ValueError("Hole diameters must be positive.")
    for pos in hole_positions:
        if len(pos) != 2:
            raise ValueError("Each hole position must be a tuple of two coordinates.")
        if not (0 <= pos[0] <= length and 0 <= pos[1] <= width):
            raise ValueError("Hole positions must be within the rectangle bounds.")
    
    # Create the rectangular base
    base = Part.makeBox(length, width, thickness)
    
    # Create and subtract each hole
    for diameter, position in zip(hole_diameters, hole_positions):
        x, y = position
        # Create a cylinder for the hole
        hole = Part.makeCylinder(diameter / 2, thickness, FreeCAD.Vector(x, y, 0), FreeCAD.Vector(0, 0, 1))
        # Subtract the hole from the base
        base = base.cut(hole)
    
    # Get the active document or create a new one if none exists
    doc = FreeCAD.activeDocument()
    if not doc:
        doc = FreeCAD.newDocument("RectangularPart")
    
    # Add the resulting shape to the document
    obj = doc.addObject("Part::Feature", "RectangularPartWithHoles")
    obj.Shape = base
    doc.recompute()
    
    return obj

# Example usage
if __name__ == "__main__":
    # Define parameters
    length = 100.0
    width = 50.0
    thickness = 10.0
    hole_diameters = [5.0, 5.0, 5.0]
    hole_positions = [(25.0, 25.0), (50.0, 25.0), (75.0, 25.0)]
    
    # Create the part
    part = create_rectangular_part_with_holes(length, width, thickness, hole_diameters, hole_positions)
    print("Rectangular part with holes created successfully.")

### chair
- [Prompt](#prompt): Provide Python code to generate a 3D CAD model of a chair, including the seat, backrest, and four legs. Include parameters for seat height, backrest angle, and leg dimensions.
```python
import FreeCAD
import Part

def create_chair_model(
    seat_width=50.0,
    seat_depth=50.0,
    seat_height=45.0,
    backrest_height=50.0,
    backrest_angle=10.0,
    leg_width=5.0,
    leg_thickness=5.0
):
    """
    Generate a 3D CAD model of a chair in FreeCAD.

    Parameters:
    - seat_width (float): Width of the seat (default: 50.0).
    - seat_depth (float): Depth of the seat (default: 50.0).
    - seat_height (float): Height from the floor to the top of the seat (default: 45.0).
    - backrest_height (float): Height of the backrest (default: 50.0).
    - backrest_angle (float): Angle of the backrest in degrees (default: 10.0).
    - leg_width (float): Width of the legs (default: 5.0).
    - leg_thickness (float): Thickness of the legs (default: 5.0).

    Returns:
    - FreeCAD object: The resulting chair model added to a new FreeCAD document.
    """
    # Create the seat as a rectangular box
    seat = Part.makeBox(seat_width, seat_depth, seat_height)

    # Create the backrest as a box, rotate it, and position it at the back of the seat
    backrest = Part.makeBox(seat_width, leg_thickness, backrest_height)
    backrest.rotate(FreeCAD.Vector(0, 0, 0), FreeCAD.Vector(0, 1, 0), backrest_angle)
    backrest.translate(FreeCAD.Vector(0, seat_depth - leg_thickness, seat_height))

    # Create the four legs as vertical boxes positioned at the corners of the seat
    leg1 = Part.makeBox(leg_width, leg_thickness, seat_height)
    leg1.translate(FreeCAD.Vector(0, 0, 0))  # Front-left leg

    leg2 = Part.makeBox(leg_width, leg_thickness, seat_height)
    leg2.translate(FreeCAD.Vector(seat_width - leg_width, 0, 0))  # Front-right leg

    leg3 = Part.makeBox(leg_width, leg_thickness, seat_height)
    leg3.translate(FreeCAD.Vector(0, seat_depth - leg_thickness, 0))  # Back-left leg

    leg4 = Part.makeBox(leg_width, leg_thickness, seat_height)
    leg4.translate(FreeCAD.Vector(seat_width - leg_width, seat_depth - leg_thickness, 0))  # Back-right leg

    # Combine all parts into a single compound object
    all_parts = [seat, backrest, leg1, leg2, leg3, leg4]
    chair = Part.makeCompound(all_parts)

    # Add the model to a new FreeCAD document
    doc = FreeCAD.newDocument("Chair")
    obj = doc.addObject("Part::Feature", "Chair")
    obj.Shape = chair
    doc.recompute()

    return obj

# Example usage
if __name__ == "__main__":
    create_chair_model()

### table
- [Prompt](#prompt): Provide Python code to generate a 3D CAD model of a table, including the tabletop, four legs, and structural support. Include parameters for tabletop dimensions, leg height, and thickness of the table surface.
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
