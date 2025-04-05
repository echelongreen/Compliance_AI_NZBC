
 Multi-Agent Building Compliance Platform

Executive Summary

AI-powered compliance verification platform that analyzes building plans before council submission, You can upload your PDF which is semi structured. dramatically reducing Request For Information (RFI) cycles and consent delays. The system leverages specialized multiple AI agents to review architectural plans, structural drawings, specifications, and council documentation against New Zealand Building Code (NZBC) and NZ Standards, providing actionable feedback with direct code references.

# Architectural Plan Processing & Compliance Verification System

Thank you for the additional details. Based on your responses, I can now outline a more specific approach for processing architectural PDFs and verifying compliance.


## Document Processing Strategy

### Step 1: PDF Decomposition and Classification
- Convert each PDF page to high-resolution images (300+ DPI)
- Classify each page into categories:
  * Architectural plans
  * Structural drawings
  * Details/sections
  * Specifications
  * Cover pages/indexes

### Step 2: Element Extraction by GPT Vision
For each page:
1. Process through GPT Vision with specialized prompts based on page type
2. Extract key elements:
   - Dimensions and measurements
   - Room designations and purposes
   - Material specifications
   - Construction methods
   - Annotations and notes
   - Scale information

### Step 3: Spatial Relationship Preservation
- Create coordinate-based references for elements
- Record cross-references between pages (e.g., "See Detail A on page 15")
- Build a relationship graph connecting related elements

### Step 4: Structured Data Representation
I recommend a hierarchical JSON structure with these advantages:
- Flexible for different element types
- Preserves spatial relationships
- Easy to query programmatically
- Can be visualized in multiple ways

Example structure:
```json
{
  "document_info": { "title": "...", "project_number": "..." },
  "pages": [
    {
      "page_number": 5,
      "type": "architectural_plan",
      "elements": [
        {
          "type": "room",
          "id": "rm_101",
          "designation": "bathroom",
          "dimensions": {"width": 2.4, "length": 3.2},
          "coordinates": {"x1": 120, "y1": 150, "x2": 360, "y2": 470},
          "references": [{"page": 15, "element_id": "det_A"}]
        },
        // more elements
      ]
    }
  ]
}
```

## Building Code Processing Strategy

### Step 1: Code Segmentation
- Break down NZBC and NZS 3604 into discrete, testable requirements
- Group requirements by building element and function

### Step 2: Create a Compliance Rule Database
- Convert each requirement into a structured format:
  ```json
  {
    "code_reference": "NZBC E2.3.2",
    "description": "Roof to wall junctions must...",
    "applies_to": ["roof_junctions", "exterior_walls"],
    "test_criteria": [
      {"parameter": "flashing_depth", "minimum": 75}
    ]
  }
  ```


### chat history 


*/ 
Favicon for anthropic
Claude 3.7 Sonnet (thinking) | Anthropic
Reasoning (thought for 19.0s)
Architectural Plan Processing & Compliance Verification System
Thank you for the additional details. Based on your responses, I can now outline a more specific approach for processing architectural PDFs and verifying compliance.

Document Processing Strategy
Step 1: PDF Decomposition and Classification

Convert each PDF page to high-resolution images (300+ DPI)

Classify each page into categories:
Architectural plans
Structural drawings
Details/sections
Specifications
Cover pages/indexes

Step 2: Element Extraction by GPT Vision
For each page:


Process through GPT Vision with specialized prompts based on page type

Extract key elements:
Dimensions and measurements
Room designations and purposes
Material specifications
Construction methods
Annotations and notes
Scale information

Step 3: Spatial Relationship Preservation

Create coordinate-based references for elements

Record cross-references between pages (e.g., "See Detail A on page 15")

Build a relationship graph connecting related elements

Step 4: Structured Data Representation
I recommend a hierarchical JSON structure with these advantages:


Flexible for different element types

Preserves spatial relationships

Easy to query programmatically

Can be visualized in multiple ways

Example structure:

json

{
  "document_info": { "title": "...", "project_number": "..." },
  "pages": [
    {
      "page_number": 5,
      "type": "architectural_plan",
      "elements": [
        {
          "type": "room",
          "id": "rm_101",
          "designation": "bathroom",
          "dimensions": {"width": 2.4, "length": 3.2},
          "coordinates": {"x1": 120, "y1": 150, "x2": 360, "y2": 470},
          "references": [{"page": 15, "element_id": "det_A"}]
        },
        // more elements
      ]
    }
  ]
}
Building Code Processing Strategy
Step 1: Code Segmentation

Break down NZBC and NZS 3604 into discrete, testable requirements

Group requirements by building element and function

Step 2: Create a Compliance Rule Database

Convert each requirement into a structured format:
json

{
  "code_reference": "NZBC E2.3.2",
  "description": "Roof to wall junctions must...",
  "applies_to": ["roof_junctions", "exterior_walls"],
  "test_criteria": [
    {"parameter": "flashing_depth", "minimum": 75}
  ]
}

Implementation Plan

Prototype Development (4-6 weeks)

Build PDF processing pipeline
Test with sample architectural plans
Implement basic GPT Vision analysis
Create sample structured output

Code Requirement Database (6-8 weeks)

Start with most common compliance issues
Create structured representations
Develop matching algorithms

Compliance Engine (8-10 weeks)

Connect extracted elements to requirements
Generate compliance reports
Highlight potential issues

UI/UX Development (concurrent)

Design user interface for document upload
Create visualization of compliance results
Implement feedback mechanism

Would you like me to elaborate on any specific part of this approach? Or should we discuss implementation technologies in more detail?







Avatar for origamiflow
origamiflow
If I upload a pdf as an example will that help you understand the structure of the PDF better?






Favicon for anthropic
Claude 3.7 Sonnet (thinking) | Anthropic
Reasoning (thought for 5.2s)
Yes, Seeing a Sample PDF Would Be Extremely Helpful
Having an actual example PDF would significantly improve my ability to give you targeted recommendations. It would help me:


Understand the specific layout patterns in your architectural documents

Identify the exact types of elements that need extraction

See how information flows between different pages and sections

Recognize potential processing challenges specific to your documents

Suggest optimized GPT Vision prompting strategies for your content

Recommend appropriate data structures based on actual content relationships

Design a more efficient processing pipeline tailored to your document format

If you're able to share a sample (even a redacted version with sensitive information removed), it would let me provide much more concrete and practical guidance for your implementation.







Avatar for origamiflow
origamiflow
This is description of document for structural plans """### Structured Output of the PDF Content (Page-by-Page Summary)

Page 1

Header: Approved Building Consent (BC 20240458) by Horowhenua District Council.

Key Details:
Address: 53 Rua Avenue, Waitarere Beach.
Engineer: Fifth Dimension Consulting Engineers.
Project No.: 24-033.
Date: September 2024.

Page 2

Concrete Work Specifications:
Minimum concrete strengths for foundations, slabs, beams, etc.
Requirements for vibration, curing, penetrations, surface finishes, and reinforcing steel.

General Notes:
Compliance with NZ Building Code and standards.
Contractor responsibilities (e.g., temporary works, subgrade verification).

Page 3

Steelwork Notes:
Fabrication standards (NZS 3404:1997).
Bolt/weld specifications, galvanization, and maintenance schedules.

Structural Maintenance Schedule:
Inspection intervals (half-yearly, 5-yearly, etc.) for steel, timber, and concrete.

Page 4

Foundation Plan (Garage Level):
Dimensions for foundation beams (FB1, FB2, FP1).
Gravel raft construction requirements.
Geotechnical notes (compaction, geogrid layers).

Page 5

Foundation Plan (Level 1):
Details for gravel raft, slab thickness, and reinforcement.
Reiteration of geotechnical requirements.

Page 6

Earthwork Sections:
Cross-sections showing gravel raft, geotextile layers, and drainage.

Pages 7–8

Anomaly: Text in Russian (appears unrelated to the rest of the document).

Page 9

Garage Floor Plan:
Precast panel details, reinforcement, and connection notes.

Page 10

Level 1 Floor Plan:
Rib-and-infill floor design, steel beams, and diaphragm specifications.

Page 11

Level 2 Framing Plan:
Timber and steel framing details (LVL joists, steel beams, connections).

Page 12

Roof Framing Plan:
Steelwork connections, proprietary trusses, and plywood diaphragm.

Pages 13–15

Building Sections:
Cross-sections showing structural steel, precast panels, and floor connections.

Pages 16–17

Foundation Details:
Reinforcement for concrete rafts, slab thickenings, and precast panel interfaces.

Pages 18–20

Structural Steel Details:
Baseplates, beam-to-column connections, and bolt specifications.

Pages 21–22

Timber/Precast Details:
Joist hangers, floor diaphragms, and connections between steel/timber.

Pages 23–24

Roof and Structural Details:
Plywood diaphragm fixing, balustrade connections, and service penetrations.

Pages 25–27

Precast Panel Elevations:
Reinforcement layouts, Drossbach ducts, and panel types (200mm/150mm).

Pages 28–32

Precast Panel Elevations (Continued):
Specific panel designs (e.g., PA-05-A, PC-01-C) with rebates, ducts, and starter bars.

Page 33

Stair Details:
Steel stringers, tread connections, and balustrade specifications.

Key Themes Across the Document:

Structural Compliance: Adherence to NZS 3101 (concrete), NZS 3404 (steel), and NZS 3604 (timber).

Material Specifications:
Concrete strengths: 35MPa (foundations), 45MPa (precast).
Steel grades: Grade 300/500E for rebar, Grade 350 for hollow sections.

Precast Panels: Detailed reinforcement, grouting ducts, and interface requirements.

Contractor Responsibilities: Verification of dimensions, temporary works, and coordination with engineers.

This structure provides a roadmap for navigating the technical requirements of the building consent document."""






Favicon for anthropic
Claude 3.7 Sonnet (thinking) | Anthropic
Reasoning (thought for 26.3s)
Insights Based on Your Structural Plans Example
Thank you for sharing this detailed description. This gives me a much clearer picture of what we're working with. Based on this structural plans example, I can offer more specific recommendations.

Document Structure Analysis
Your structural plans have a logical organization with several distinct section types:


Administrative information (page 1)

General specifications and requirements (pages 2-3)

Level-specific plans (foundation, floor, framing, roof)

Cross-sectional details

Component-specific details (foundation, steel, precast, timber)

Refined Processing Approach
1. Page Classification System
First, we'll need to categorize each page:


Cover/administrative pages: Extract project details, consent numbers

Specification pages: Extract material properties, standards references

Plan pages: Extract dimensions, structural elements, references

Detail pages: Extract connection types, fasteners, reinforcement

Elevation pages: Extract panel types, openings, reinforcement

2. GPT Vision Implementation
I recommend a multi-pass approach:

First Pass:

python

# Pseudocode for first pass
for page_image in pdf_images:
    # Send to GPT Vision with classification prompt
    page_type = gpt_vision_analyze(
        page_image, 
        prompt="Identify the type of this architectural/structural drawing page. Is this a cover page, general notes, plan view, section, detail, or elevation? What level does it represent? Extract any page numbers or references."
    )
    
    # Store classification
    document_structure[page_number] = {
        "type": page_type,
        "level": detected_level,
        "references": detected_references
    }
Second Pass:

python

# Pseudocode for second pass - targeted extraction
for page_number, page_info in document_structure.items():
    # Use type-specific prompts
    if page_info["type"] == "plan_view":
        prompt = "Identify all structural elements (beams, columns, foundations), their dimensions, materials, and any callouts or references to other pages."
    elif page_info["type"] == "specification":
        prompt = "Extract all material specifications, compliance standards, and maintenance requirements in a structured format."
    # ...etc
    
    # Extract detailed information
    page_details = gpt_vision_analyze(pdf_images[page_number], prompt)
    document_structure[page_number]["elements"] = page_details
3. Structured Data Format
Based on your specific document, I recommend this JSON structure:

json

{
  "document_info": {
    "title": "Structural Plans",
    "project_number": "24-033",
    "address": "53 Rua Avenue, Waitarere Beach",
    "consent_number": "BC 20240458"
  },
  "material_specifications": {
    "concrete": [
      {"type": "foundations", "strength": "35MPa", "standard": "NZS 3101"}
    ],
    "steel": [
      {"type": "reinforcement", "grade": "300/500E"}
    ]
  },
  "plans": {
    "foundation": {
      "garage_level": {
        "elements": [
          {
            "type": "foundation_beam",
            "id": "FB1",
            "dimensions": {"width": "X", "depth": "Y"},
            "references": {"page": 16, "detail": "foundation_detail_1"}
          }
        ]
      }
    }
  },
  "compliance_references": [
    {"standard": "NZS 3101", "section": "X.X.X", "pages": [2, 3]}
  ]
}
4. Compliance Database Structure
For the standards database, I suggest organizing it hierarchically:

json

{
  "NZS_3604": {
    "foundations": {
      "concrete_strength": {
        "requirement": "Minimum 17.5MPa at 28 days",
        "section": "4.5.2",
        "applies_to": ["foundation_walls", "footings"]
      },
      "dimensions": {
        "requirement": "Minimum width of 150mm",
        "section": "4.4.1",
        "applies_to": ["foundation_walls"]
      }
    }
  }
}
Would you like me to focus on any specific aspect of this processing approach? I can elaborate on the GPT Vision prompting strategy, the data structure design, or the implementation workflow. 