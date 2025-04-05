# Stage 1: Architectural Document Processing Pipeline

## Overview

This stage focuses on converting PDF architectural drawings to structured data through a systematic page-by-page processing approach using GPT Vision. The pipeline transforms unstructured visual information into a standardized schema ready for NZBC compliance checking.

## Step-by-Step Implementation Process

### Step 1: Document Preparation & Image Conversion

```python
def prepare_document(pdf_path):
    """Convert PDF to high-resolution images ready for processing."""
    
    # Create processing directory
    process_id = generate_unique_id()
    process_dir = f"processing/{process_id}"
    os.makedirs(process_dir, exist_ok=True)
    
    # Convert PDF to images using pdf2image
    images = convert_from_path(
        pdf_path, 
        dpi=300,  # High resolution for detail capture
        thread_count=4,  # Parallel processing
        fmt="png"
    )
    
    # Save images and create manifest
    image_paths = []
    for i, image in enumerate(images):
        image_path = f"{process_dir}/page_{i+1:03d}.png"
        image.save(image_path)
        image_paths.append(image_path)
    
    # Create processing manifest
    manifest = {
        "process_id": process_id,
        "original_filename": os.path.basename(pdf_path),
        "page_count": len(images),
        "image_paths": image_paths,
        "creation_timestamp": datetime.now().isoformat(),
        "status": "ready_for_processing"
    }
    
    with open(f"{process_dir}/manifest.json", "w") as f:
        json.dump(manifest, f, indent=2)
    
    return process_id, manifest
```

### Step 2: Page Classification

```python
def classify_page(image_path):
    """Determine page type and extract metadata using GPT Vision."""
    
    # Load and prepare image
    image = Image.open(image_path)
    
    # Create classification prompt
    prompt = """
    Analyze this architectural drawing and identify the following:
    1. Document type (administrative, elevation, site_plan, structural, floor_plan, services, material_specification)
    2. Building level (foundation, ground_floor, first_floor, roof, site)
    3. View type (plan, elevation, section, isometric, 3d)
    4. Page metadata (drawing number, scale, title)
    5. Presence of gridline system
    
    Format your response as JSON.
    """
    
    # Query GPT Vision API
    response = query_gpt_vision(image, prompt)
    classification = json.loads(response)
    
    # Enhance metadata extraction for title blocks
    if "title_block_detected" in classification and classification["title_block_detected"]:
        title_block = extract_title_block(image)
        classification.update(title_block)
    
    return classification
```

### Step 3: Type-Specific Content Extraction

```python
def extract_page_content(image_path, page_classification):
    """Extract structured content based on page classification."""
    
    image = Image.open(image_path)
    page_type = page_classification["document_type"]
    
    # Select appropriate extraction function based on page type
    if page_type == "administrative":
        extraction_result = extract_administrative_content(image)
    elif page_type == "elevation":
        extraction_result = extract_elevation_content(image, page_classification)
    elif page_type == "site_plan":
        extraction_result = extract_site_plan_content(image)
    elif page_type == "structural":
        extraction_result = extract_structural_content(image, page_classification)
    elif page_type == "floor_plan":
        extraction_result = extract_floor_plan_content(image, page_classification)
    elif page_type == "services":
        extraction_result = extract_services_content(image)
    elif page_type == "material_specification":
        extraction_result = extract_material_content(image)
    else:
        extraction_result = extract_general_content(image)
        
    # Add page classification metadata to result
    extraction_result["document_metadata"] = page_classification
    
    return extraction_result
```

### Step 4: Type-Specific Extraction Implementation

Each type of drawing requires a specialized extraction function with custom prompts:

#### Example: Floor Plan Extraction

```python
def extract_floor_plan_content(image, classification):
    """Extract structured data from floor plans."""
    
    level = classification.get("building_level", "unknown")
    
    prompt = f"""
    Analyze this {level} floor plan drawing and extract the following elements:
    
    1. All rooms with names, dimensions, and areas
    2. Wall types, thicknesses, and materials
    3. Door and window openings with dimensions
    4. Circulation spaces and clearances
    5. Any compliance-related notes or annotations
    6. Gridline references where present
    
    Format your response as structured JSON according to this schema:
    {{
      "spatial_elements": {{
        "rooms": [
          {{
            "name": "kitchen|living|bathroom|bedroom",
            "dimensions": {{"width": "3.2m", "length": "4.5m"}},
            "area": "14.4m²",
            "grid_location": "C3-D5",
            "floor_level": "{level}"
          }}
        ],
        "circulation": [
          {{
            "type": "hallway|stairway|entrance",
            "width": "1200mm",
            "compliance_notes": "description"
          }}
        ],
        "openings": [
          {{
            "type": "door|window|access_panel",
            "dimensions": {{"width": "900mm", "height": "2100mm"}},
            "grid_location": "B4"
          }}
        ]
      }},
      "compliance_information": {{
        "notes": [
          {{
            "text": "note text",
            "related_code": "code reference if mentioned"
          }}
        ]
      }}
    }}
    """
    
    # Query GPT Vision API
    response = query_gpt_vision(image, prompt)
    
    # Parse and validate response
    try:
        extracted_data = json.loads(response)
        validate_floor_plan_schema(extracted_data)
        return extracted_data
    except Exception as e:
        return {
            "error": str(e),
            "raw_response": response,
            "spatial_elements": {"rooms": [], "circulation": [], "openings": []}
        }
```

#### Example: Structural Drawing Extraction

```python
def extract_structural_content(image, classification):
    """Extract structured data from structural drawings."""
    
    level = classification.get("building_level", "unknown")
    
    prompt = f"""
    Analyze this structural drawing for the {level} and extract:
    
    1. Complete gridline system (horizontal and vertical references)
    2. All structural elements (beams, columns, walls, footings, etc.)
    3. Materials and dimensions of structural elements
    4. Reinforcement specifications
    5. Structural notes and compliance requirements
    
    Format your response as structured JSON according to this schema:
    {{
      "structural_elements": {{
        "gridlines": {{
          "horizontal": ["1", "2", "3", "..."],
          "vertical": ["A", "B", "C", "..."]
        }},
        "foundation": [
          {{
            "type": "footing|slab|pile",
            "dimensions": {{"width": "600mm", "depth": "300mm"}},
            "grid_location": "B3-D3",
            "material": "concrete",
            "reinforcement": "D16@200crs"
          }}
        ],
        "walls": [
          {{
            "type": "external|internal|load_bearing",
            "thickness": "140mm",
            "material": "timber|concrete|masonry",
            "grid_location": "A1-A4"
          }}
        ],
        "beams_columns": [
          {{
            "type": "beam|column|brace",
            "id": "B1",
            "dimensions": {{"width": "90mm", "depth": "350mm"}},
            "material": "timber|steel|concrete",
            "grid_location": "A1-A4"
          }}
        ]
      }},
      "compliance_information": {{
        "notes": [
          {{
            "text": "note text",
            "related_code": "code reference if mentioned"
          }}
        ]
      }}
    }}
    """
    
    # Query GPT Vision API
    response = query_gpt_vision(image, prompt)
    
    # Parse and validate response
    try:
        extracted_data = json.loads(response)
        validate_structural_schema(extracted_data)
        return extracted_data
    except Exception as e:
        return {
            "error": str(e),
            "raw_response": response,
            "structural_elements": {
                "gridlines": {"horizontal": [], "vertical": []},
                "foundation": [], 
                "walls": [], 
                "beams_columns": []
            }
        }
```

### Step 5: Schema Validation & Normalization

```python
def validate_and_normalize_extraction(extracted_data, page_classification):
    """Validate extraction against schema and normalize values."""
    
    document_type = page_classification["document_type"]
    schema_validator = get_schema_validator(document_type)
    
    # Validate schema
    validation_errors = schema_validator.validate(extracted_data)
    if validation_errors:
        extracted_data["validation_errors"] = validation_errors
    
    # Normalize measurements
    extracted_data = normalize_measurements(extracted_data)
    
    # Normalize material references
    extracted_data = normalize_material_references(extracted_data)
    
    # Add confidence scores for extracted elements
    extracted_data = add_confidence_scores(extracted_data)
    
    return extracted_data
```

### Step 6: Document-wide Coordination

```python
def coordinate_document_elements(processed_pages):
    """Create cross-references between pages and coordinate element IDs."""
    
    # Extract gridline systems from pages
    gridline_systems = {}
    for page_id, page_data in processed_pages.items():
        if "structural_elements" in page_data and "gridlines" in page_data["structural_elements"]:
            gridline_systems[page_id] = page_data["structural_elements"]["gridlines"]
    
    # Unify gridline system
    unified_gridlines = unify_gridline_systems(gridline_systems)
    
    # Update grid references
    for page_id, page_data in processed_pages.items():
        if "structural_elements" in page_data:
            page_data["structural_elements"]["gridlines"] = unified_gridlines
            
    # Coordinate material references
    material_registry = {}
    for page_id, page_data in processed_pages.items():
        extract_material_references(page_data, material_registry)
    
    # Unify material references across pages
    for page_id, page_data in processed_pages.items():
        page_data = update_material_references(page_data, material_registry)
        processed_pages[page_id] = page_data
    
    # Add cross-page references
    processed_pages = generate_cross_references(processed_pages)
    
    return processed_pages
```

### Step 7: Comprehensive Processing Pipeline

```python
def process_document(pdf_path):
    """Process entire document from PDF to structured data."""
    
    # Step 1: Prepare document and convert to images
    process_id, manifest = prepare_document(pdf_path)
    
    # Step 2: Process each page
    processed_pages = {}
    for image_path in manifest["image_paths"]:
        page_number = int(os.path.basename(image_path).split("_")[1].split(".")[0])
        
        # Classify page
        classification = classify_page(image_path)
        
        # Extract content based on classification
        extraction_result = extract_page_content(image_path, classification)
        
        # Validate and normalize extraction
        normalized_data = validate_and_normalize_extraction(extraction_result, classification)
        
        # Store processed result
        processed_pages[page_number] = normalized_data
        
        # Update processing status
        update_processing_status(process_id, f"Processed page {page_number}/{manifest['page_count']}")
    
    # Step 3: Coordinate document-wide elements
    coordinated_data = coordinate_document_elements(processed_pages)
    
    # Step 4: Save final processed data
    save_processed_data(process_id, coordinated_data)
    
    # Step 5: Generate processing report
    report = generate_processing_report(process_id, coordinated_data)
    
    return process_id, report
```

## Comprehensive Data Schema

The core schema for storing the extracted data:

```json
{
  "document_metadata": {
    "page_id": "A1.XX|A2.XX",
    "document_type": "administrative|elevation|site_plan|structural|floor_plan|services|material_specification",
    "building_level": "foundation|ground_floor|first_floor|roof|site",
    "view_type": "plan|elevation|section|isometric|3d",
    "view_direction": "north|south|east|west",
    "scale": "1:50|1:100|1:200",
    "drawing_series": "A1|A2",
    "title": "Drawing title text",
    "revision": "Revision information",
    "date": "YYYY-MM-DD"
  },
  
  "spatial_elements": {
    "rooms": [
      {
        "name": "kitchen|living|bathroom|bedroom",
        "dimensions": {"width": "3.2m", "length": "4.5m"},
        "area": "14.4m²",
        "grid_location": "C3-D5",
        "floor_level": "ground_floor|first_floor"
      }
    ],
    "circulation": [
      {
        "type": "hallway|stairway|entrance",
        "width": "1200mm",
        "compliance_notes": "Minimum 1000mm required for accessibility"
      }
    ],
    "openings": [
      {
        "type": "door|window|access_panel",
        "dimensions": {"width": "900mm", "height": "2100mm"},
        "grid_location": "B4",
        "elevation_reference": "north|south|east|west"
      }
    ]
  },
  
  "structural_elements": {
    "gridlines": {
      "horizontal": ["1", "2", "3", "4", "5", "6", "7", "8", "9", "10"],
      "vertical": ["A", "B", "C", "D", "E", "F", "G", "H"]
    },
    "foundation": [
      {
        "type": "footing|slab|pile",
        "dimensions": {"width": "600mm", "depth": "300mm"},
        "grid_location": "B3-D3",
        "material": "concrete",
        "reinforcement": "D16@200crs"
      }
    ],
    "flooring_system": [
      {
        "type": "joist|bearer|blocking",
        "dimensions": {"width": "90mm", "depth": "240mm"},
        "spacing": "400mm",
        "material": "timber|lvl|steel",
        "direction": "north-south|east-west",
        "span": "3.6m",
        "grid_location": "B3-B7"
      }
    ],
    "walls": [
      {
        "type": "external|internal|load_bearing",
        "thickness": "140mm",
        "material": "timber|concrete|masonry",
        "grid_location": "A1-A4"
      }
    ],
    "beams_columns": [
      {
        "type": "beam|column|brace",
        "id": "B1",
        "dimensions": {"width": "90mm", "depth": "350mm"},
        "material": "timber|steel|concrete",
        "grid_location": "A1-A4"
      }
    ]
  },
  
  "elevation_elements": {
    "building_heights": [
      {
        "measurement_point": "roof_peak|eave|parapet|floor_level",
        "height": "7.2m",
        "relative_to": "ground_level|datum",
        "compliance_note": "Maximum allowed height 8.0m"
      }
    ],
    "material_zones": [
      {
        "zone_id": "EZ1",
        "material": "brick|timber|metal|fiber_cement",
        "location": "north_wall|west_wall_upper",
        "dimensions": {"height": "2.4m", "width": "5.6m"},
        "color_on_drawing": "blue|yellow|green"
      }
    ],
    "ground_profile": [
      {
        "type": "existing_ground|finished_ground",
        "points": [
          {"x": 0, "y": 0, "elevation": "100.0m"},
          {"x": 5, "y": 0, "elevation": "100.5m"}
        ]
      }
    ]
  },
  
  "site_elements": {
    "boundaries": [
      {
        "type": "property_line|setback|easement",
        "compliance_status": "compliant|infringement",
        "highlighted": true,
        "dimensions": {"distance": "3.0m"},
        "compliance_notes": "Yard infringement requires consent"
      }
    ],
    "earthworks": [
      {
        "type": "cut|fill",
        "volume": "XX m³",
        "maximum_depth": "1.2m",
        "compliance_notes": "Requires resource consent"
      }
    ]
  },
  
  "services": {
    "plumbing": [
      {
        "type": "supply|waste|vent",
        "material": "copper|pvc|pex",
        "location": "wall|ceiling|floor",
        "compliance_notes": "As per G13 requirements"
      }
    ],
    "electrical": [
      {
        "type": "lighting|power|data",
        "compliance_notes": "As per G9 requirements"
      }
    ],
    "hvac": [
      {
        "type": "duct|diffuser|unit",
        "dimensions": {"width": "300mm", "height": "200mm"},
        "compliance_notes": "As per G4 requirements"
      }
    ]
  },
  
  "materials": {
    "exterior_finishes": [
      {
        "id": "M1",
        "type": "cladding|roofing|window|door",
        "material": "brick|timber|metal|fiber_cement|plaster",
        "finish": "painted|stained|powder_coated",
        "color": "color_description",
        "compliance_reference": "E2_EXTERNAL_MOISTURE"
      }
    ],
    "structural_materials": [
      {
        "id": "SM1",
        "type": "concrete|steel|timber|masonry",
        "grade": "material_grade",
        "specification": "material_standard_reference",
        "compliance_reference": "B1_STRUCTURE"
      }
    ]
  },
  
  "compliance_information": {
    "building_code_references": [
      {
        "code": "B1_STRUCTURE|E2_EXTERNAL_MOISTURE|G4_VENTILATION",
        "clause": "X.X.X",
        "requirement": "Description of requirement",
        "compliance_status": "compliant|non_compliant|requires_verification"
      }
    ],
    "highlighted_areas": [
      {
        "description": "accessibility clearance zone|yard infringement",
        "color": "yellow|red",
        "approval_status": "pending|approved|requires_consent"
      }
    ],
    "consent_requirements": [
      {
        "type": "resource_consent|building_consent",
        "status": "approved|pending|required",
        "issuing_authority": "Horowhenua District Council"
      }
    ]
  },
  
  "cross_references": {
    "related_drawings": [
      {
        "source_element": "beam_B1",
        "target_page": "A2.05",
        "relationship": "detail|section|elevation"
      }
    ],
    "vertical_relationships": [
      {
        "element": "wall_W1",
        "related_elements": ["foundation_F3", "framing_J12"]
      }
    ]
  },
  
  "processing_metadata": {
    "confidence_score": 0.85,
    "processing_timestamp": "2023-06-15T14:30:22Z",
    "validation_errors": [],
    "extraction_warnings": []
  }
}
```

## Implementation Considerations

1. **Parallel Processing**:
   - Process multiple pages simultaneously to improve throughput
   - Use task queues (e.g., Celery) for distributed processing

2. **Error Handling**:
   - Implement robust validation to catch extraction issues
   - Store raw GPT responses for debugging
   - Track confidence scores for extracted elements

3. **Performance Optimization**:
   - Cache similar document types to optimize GPT Vision usage
   - Use lower resolution for classification, higher for detailed extraction
   - Batch process images during low-demand periods

4. **Cost Management**:
   - Optimize image size for GPT Vision API
   - Implement request throttling and batching
   - Cache common document patterns

By following this processing pipeline, you can systematically transform architectural drawings into structured data ready for NZBC compliance checking.