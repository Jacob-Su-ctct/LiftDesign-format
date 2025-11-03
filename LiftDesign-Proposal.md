# LiftDesign Format Proposal

The LiftLayer output format specification is built upon the LandXML-1.2 schema. This format is necessary for sharing the design with other machines or systems and must support signature validation.

## LiftLayer Output Format Specification (LandXML-1.2 based)

The file typically utilizes the `<Application>` element to define the design parameters and the `<Surfaces>` element to define the composite surface and metadata references. The tool generating this output is named "CTCT_LifLayer_Generator".

### 1. Application Level

The top-level design metadata is stored in an `<Application>` feature named "LiftDesign".

| Property Label | XML Path Example | Description | Source Citation |
|----------------|------------------|-------------|-----------------|
| Feature Name | `Application/Feature[@name="LiftDesign"]` | Defines the design document metadata. | |
| document_uuid | `Property[@label="document_uuid"]` | Unique identifier for the LiftLayer design document. | |
| design_name | `Property[@label="design_name"]` | Human-readable name for the lift layers design. | |

### 2. Surface Metadata

Information regarding the source surfaces (Critical, Cut, Fill) used to build the final composite surface is stored within the `<Surfaces>` element under a `<Feature>` named "metadata".

| Property Label | XML Path Example | Description | Source Citation |
|----------------|------------------|-------------|-----------------|
| Feature Name | `Surfaces/Feature[@name="metadata"]` | Defines references and configuration for the component surfaces. | |
| critical_name | `Property[@label="critical_name"]` | Name of the Critical Surface reference. | |
| critical_uuid | `Property[@label="critical_uuid"]` | Unique identifier (UUID) for the Critical Surface. | |
| cut_name | `Property[@label="cut_name"]` | Name of the Cut Surface reference. | |
| cut_uuid | `Property[@label="cut_uuid"]` | Unique identifier (UUID) for the Cut Surface. | |
| cut_offset | `Property[@label="cut_offset"]` | Offset value applied to the cut surface. Separate offsets are required for cut and fill surfaces. | |
| fill_name | `Property[@label="fill_name"]` | Name of the Fill Surface reference. | |
| fill_uuid | `Property[@label="fill_uuid"]` | Unique identifier (UUID) for the Fill Surface. | |
| fill_offset | `Property[@label="fill_offset"]` | Offset value applied to the fill surface. | |
| smoothing | `Property[@label="smoothing"]` | Overall smoothing percentage applied to the design, often applied to surface transitions. | |

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

### 3. Face Classifications

Within the composite surface definition (using `surfType="TIN"`), specific `<Feature>` tags are inserted into the `<Faces>` element to classify the origin of each triangular face (`<F>`).

| Feature Name | Description | Related Properties (Examples) | Source Citation |
|--------------|-------------|------------------------------|-----------------|
| cut_faces | Classifies triangular faces belonging to the cut portion of the composite surface. | offset: "0.5m" | |
| critical_faces | Classifies triangular faces belonging to the critical portion of the composite surface. | offset: "0.3m"; smoothing: "0.8" | |
| fill_faces | Classifies triangular faces belonging to the fill portion of the composite surface. | offset: "0.15m" | |

---

This specification details the structure for defining the lift layers within LandXML.
