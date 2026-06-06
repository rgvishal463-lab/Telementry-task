# Data Model Unification Algorithm

## Overview
This algorithm provides a systematic approach to unify two or more different data models into a single standardized format. It uses format detection, field mapping, and data transformation to normalize diverse data structures.

---

## Problem Statement
When integrating data from multiple sources, we often encounter different representations of the same logical entity:
- **Format 1**: Flat structure with concatenated strings
- **Format 2**: Nested structure with different naming conventions
- **Unified Model**: Standardized canonical representation

### Example Use Case
Industrial IoT devices reporting status in different formats need to be unified for consistent processing.

---

## Algorithm Steps

### Step 1: Format Detection
**Purpose**: Identify which format the input data follows

```
Input: JSON Object
├─ Check for format-specific markers
├─ Format 1: Contains 'deviceID' and 'location' (string)
├─ Format 2: Contains 'device' (object) and ISO 8601 timestamp
└─ Return: Format identifier
```

**Implementation**:
```python
def detectFormat(jsonObject):
    # Check for unique identifiers of each format
    if jsonObject.get('device') is not None:
        return 'FORMAT_2'
    elif jsonObject.get('deviceID') is not None:
        return 'FORMAT_1'
    else:
        return 'UNKNOWN'
```

**Complexity**: O(1) - Constant time lookup

---

### Step 2: Schema Mapping
**Purpose**: Define how fields from source format map to target format

#### Format 1 Schema Map
```
Source Field          → Target Field
────────────────────────────────────
deviceID              → deviceID
deviceType            → deviceType
timestamp             → timestamp
location (string)     → location (object) [requires split]
operationStatus       → data.status
temp                  → data.temperature
```

#### Format 2 Schema Map
```
Source Field          → Target Field
────────────────────────────────────
device.id             → deviceID
device.type           → deviceType
timestamp (ISO 8601)  → timestamp [requires conversion]
country               → location.country
city                  → location.city
area                  → location.area
factory               → location.factory
section               → location.section
data                  → data (direct copy)
```

---

### Step 3: Data Transformation
**Purpose**: Apply format-specific transformations

#### Transformation Rules

**For Format 1**:
1. **String Parsing**: Split location path
   ```
   Input:  "japan/tokyo/keiyō-industrial-zone/daikibo-factory-meioy/section-1"
   Output: {country, city, area, factory, section}
   ```

2. **Field Renaming**: Map legacy fields to standard names
   ```
   operationStatus → status
   temp → temperature
   ```

**For Format 2**:
1. **Timestamp Conversion**: ISO 8601 → Milliseconds since epoch
   ```
   Input:  "2021-06-23T10:57:17.783Z"
   Process: Parse → Convert to UTC → Multiply by 1000
   Output: 1624445837783
   ```

2. **Object Flattening**: Extract nested device info
   ```
   device.id → deviceID
   device.type → deviceType
   ```

---

### Step 4: Unified Format Construction
**Purpose**: Build the standardized output object

```
Unified Schema:
{
  deviceID: string,
  deviceType: string,
  timestamp: number (milliseconds since epoch),
  location: {
    country: string,
    city: string,
    area: string,
    factory: string,
    section: string
  },
  data: {
    status: string,
    temperature: number
  }
}
```

---

## Algorithm Pseudocode

```
FUNCTION UnifyDataModels(inputObject)
  // Step 1: Detect format
  format = DetectFormat(inputObject)
  
  IF format == UNKNOWN THEN
    THROW "Unsupported format"
  END IF
  
  // Step 2: Apply format-specific transformation
  IF format == FORMAT_1 THEN
    unifiedObject = TransformFormat1(inputObject)
  ELSE IF format == FORMAT_2 THEN
    unifiedObject = TransformFormat2(inputObject)
  END IF
  
  // Step 3: Validate output
  IF NOT ValidateUnifiedFormat(unifiedObject) THEN
    THROW "Validation failed"
  END IF
  
  // Step 4: Return unified object
  RETURN unifiedObject
END FUNCTION
```

---

## Complexity Analysis

| Aspect | Complexity | Details |
|--------|-----------|---------|
| **Time** | O(1) | Fixed number of fields to process |
| **Space** | O(1) | Output size independent of input size |
| **Format Detection** | O(1) | Constant key lookups |
| **String Parsing** | O(k) | k = number of path segments (typically 5) |
| **Timestamp Conversion** | O(1) | DateTime parsing is linear in timestamp format length |

**Overall**: O(1) - All operations are constant time

---

## Implementation Strategy

### 1. Modular Design
```
UnificationEngine
├── FormatDetector
│   └── detectFormat()
├── SchemaMapper
│   ├── FORMAT_1_SCHEMA
│   └── FORMAT_2_SCHEMA
├── Transformers
│   ├── Format1Transformer
│   │   ├── parseLocation()
│   │   └── renameFields()
│   └── Format2Transformer
│       ├── convertTimestamp()
│       └── flattenDevice()
└── Validator
    └── validateUnifiedFormat()
```

### 2. Error Handling
```python
try:
    unifiedObject = main(inputObject)
except FormatDetectionError:
    log("Unknown input format")
except TransformationError:
    log("Transformation failed for detected format")
except ValidationError:
    log("Output validation failed")
```

### 3. Extensibility
To add support for Format 3:
1. Add detection marker in `detectFormat()`
2. Define FORMAT_3_SCHEMA in SchemaMapper
3. Implement Format3Transformer
4. Update main() function with new condition
5. Add unit tests for Format 3

---

## Advantages

✅ **Scalable**: Easy to add new formats  
✅ **Maintainable**: Clear separation of concerns  
✅ **Efficient**: O(1) complexity  
✅ **Testable**: Each component can be tested independently  
✅ **Flexible**: Schema can be configured externally  
✅ **Type-Safe**: Can be enhanced with type checking  

---

## Use Cases

1. **IoT Data Aggregation**: Unify sensor data from different manufacturers
2. **Multi-Source Analytics**: Combine data from various APIs
3. **Legacy System Integration**: Bridge old and new data formats
4. **Data Migration**: Transform data during system upgrades
5. **ETL Pipelines**: Normalize data in Extract-Transform-Load processes

---

## Validation Checklist

Before marking data as unified, verify:

- [ ] All required fields are present
- [ ] Timestamp is in milliseconds since epoch (number)
- [ ] Location object has all 5 required sub-fields
- [ ] Device IDs are non-empty strings
- [ ] Status values are valid
- [ ] Temperature values are numbers

---

## Example Flow

```
Input: {"deviceID": "dh28dslkja", "location": "japan/tokyo/...", ...}
       ↓
Step 1: Format = FORMAT_1
       ↓
Step 2: Schema mapping applied
       ↓
Step 3: String split, field renaming
       ↓
Step 4: Output validation
       ↓
Output: {"deviceID": "...", "location": {nested}, "data": {...}}
```

---

## Testing Strategy

### Unit Tests
- Test each transformer independently
- Test format detection for edge cases
- Test validation rules

### Integration Tests
- Test end-to-end flow for each format
- Test mixed format batches
- Test error handling

### Performance Tests
- Measure transformation time
- Verify O(1) complexity
- Test with large batches

---

## Conclusion

This algorithm provides a robust, scalable framework for unifying multiple data models. By separating concerns into detection, mapping, transformation, and validation phases, it remains maintainable and extensible for future requirements.
