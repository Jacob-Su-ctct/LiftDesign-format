# LiftDesign Format Proposal v0.3

The LiftLayer output format specification is built upon the LandXML-1.2 schema. This format is necessary for sharing the design with other machines or systems and must support signature validation.

## LiftLayer Output Format Specification (LandXML-1.2 based)

The file utilizes the `<Project>` element to define the design parameters and surface metadata references, and the `<Surfaces>` element to define the composite surface data. The tool generating this output is named "CTCT_LifLayer_Generator".

### 1. Project Level

The top-level design metadata and surface references are stored in a `<Project>` element containing a feature with code "infield.surface".

| Property Label | XML Path Example | Description |
|----------------|------------------|-------------|
| Feature Code | `Project/Feature[@code="infield.surface"]` | Defines the design document metadata and project configuration. |
| type | `Property[@label="type"]` | Specifies the file format type identifier. |
| version | `Property[@label="version"]` | Version of the LiftLayer format specification. |
| editable | `Property[@label="editable"]` | Flag indicating if the design is editable (1=true, 0=false). If editable, the source surfaces must be included in the design file |
| project_uuid | `Property[@label="project_uuid"]` | Unique identifier for the project. |
| created_date | `Property[@label="created_date"]` | Date when the project was created. |
| last_modified | `Property[@label="last_modified"]` | Date when the project was last modified. |
| critical_name | `Property[@label="critical_name"]` | Name of the Critical Surface reference. |
| critical_uuid | `Property[@label="critical_uuid"]` | Unique identifier (UUID) for the Critical Surface. |
| cut_name | `Property[@label="cut_name"]` | Name of the Cut Surface reference. |
| cut_uuid | `Property[@label="cut_uuid"]` | Unique identifier (UUID) for the Cut Surface. |
| cut_offset | `Property[@label="cut_offset"]` | Offset value applied to the cut surface. Separate offsets are required for cut and fill surfaces. |
| fill_name | `Property[@label="fill_name"]` | Name of the Fill Surface reference. |
| fill_uuid | `Property[@label="fill_uuid"]` | Unique identifier (UUID) for the Fill Surface. |
| fill_offset | `Property[@label="fill_offset"]` | Offset value applied to the fill surface. |
| smoothing | `Property[@label="smoothing"]` | Overall smoothing percentage applied to the design, often applied to surface transitions. |
| layer_number | `Property[@label="layer_number"]` | Total number of composite surfaces when generated. |

### 2. Application Level (optional)

The `<Application>` element identifies the tool and its version that generated the LiftLayer output but no longer contains the design metadata (moved to Project level).

### 3. Surface Definitions

The `<Surfaces>` element contains the composite surface definitions and the source surfaces (Critical, Cut, Fill) used to build the final composite surfaces. Each surface includes geometric data (points and triangular faces) and optional feature classifications.

#### Composite Surface Generation Logic

The referenced critical, cut, and fill surfaces are merged into a single composite surface using conditional rules:

```
IF (Cut > Critical) 
    Composite = Cut
ELSEIF (Fill < Critical) 
    Composite = Fill
ELSE 
    Composite = Critical
END
```

This logic dictates which input surface elevation is used for the composite surface at any given point.

### 4. Face Classifications (optional)

Within the composite surface definition (using `surfType="TIN"`), specific `<Feature>` tags are inserted into the `<Faces>` element to classify the origin of each triangular face (`<F>`). These features group faces by their source surface and can include additional properties specific to each face type.

#### Face Feature Structure

Each face classification follows this XML structure:

```xml
<!-- optional: [surface type] part of the composite surface -->
<Feature name="[face_type]">
  <Property label="[property_name]" value="[property_value]"/>
  <!-- Additional properties as needed -->
</Feature>
```

| Feature Name | Description | Related Properties | Example XML |
|--------------|-------------|-------------------|-------------|
| cut_faces | Classifies triangular faces belonging to the cut portion of the composite surface. | offset: Applied cut offset value | `<Feature name="cut_faces"><Property label="offset" value="0.5m"/></Feature>` |
| critical_faces | Classifies triangular faces belonging to the critical portion of the composite surface. | offset: Applied offset value<br>smoothing: Smoothing factor applied | `<Feature name="critical_faces"><Property label="offset" value="0.3m"/><Property label="smoothing" value="0.8"/></Feature>` |
| fill_faces | Classifies triangular faces belonging to the fill portion of the composite surface. | offset: Applied fill offset value | `<Feature name="fill_faces"><Property label="offset" value="0.15m"/></Feature>` |

#### Face Grouping

The triangular faces (`<F>`) that follow each `<Feature>` tag belong to that classification until the next `<Feature>` tag is encountered or the `<Faces>` section ends. This allows for flexible grouping of faces based on their source surface and properties.

---

This specification details the structure for defining the lift layers within LandXML.
