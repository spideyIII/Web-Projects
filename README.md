# Web-Projects
Based on the current U035 screen/formula sources plus the three attached MOD PDFs, the DRF math logic does not appear to change by MOD; it changes by the DRF Type on the selected parent row. The child-row data shape is consistent across MOD 01, 70, and 80, and the trigger for recalculation is user edits to the measurement inputs—primarily B and C, with Amount Below Dwg Min only applicable for one branch. 
 
 
 

1. Confirmed U035 DRF child columns
From the current U035 formulas, the DRF detail rows use this child schema:

InspectionDRFResultId

ParentId

DefectLocation

DefectDescription

FaceDefectOnly

DepthB

MinimumWallC

WallCalculation

AdjacentWallThickness

AmountBelowMin

MeasurementMethod

Notes 

Those same fields are loaded from and saved back to 'U035 Inspection DRF Results', including:

'Depth B'

'Minimum Wall Thickness C'

'Wall Calculation'

'Adjacent Unaffected Wall Thickness'

'Amount Below Dwg Min'

'Measurement Method' 

2. The actual switch that changes the formula
The logic is driven by the selected parent row’s DRFType, not the ship MOD itself. The current screen YAML explicitly labels the calc column using:

if DRFType = "A" → "% Wall Reduction B/C X 100"

else → "Actual Wall Remaining 'A' ('C'-'B')" 

The current U035 edit formula also switches on varDRF.DRFType when recalculating WallCalculation after C changes:

Type A: (B / C) * 100

Type B: C - B 

So the real trigger is:

Trigger field: DRFType on the parent record
Trigger values: "A" or "B"
Effect: chooses which WallCalculation formula to run. 

3. The DRF calculation logic
Type A DRFs
Use percent wall reduction:

WallCalculation =
If(
    MinimumWallC <> 0,
    (DepthB / MinimumWallC) * 100,
    0
)
Meaning:

B = measured defect depth or wall reduction

C = design/minimum/adjacent unaffected thickness as used by the form

result = percent reduction. 

Type B DRFs
Use actual wall remaining:

WallCalculation =
MinimumWallC - DepthB
Meaning:

subtract defect depth/reduction B from baseline thickness C

result is remaining wall. 

4. The visibility / applicability change
AmountBelowMin is not universally used. The save formula only writes 'Amount Below Dwg Min' when the row’s parent DRFType = "B"; otherwise it saves blank. 

So the practical rule is:

Type A

show/use DepthB

show/use MinimumWallC

compute % Wall Reduction

AmountBelowMin not needed / blank on save

Type B

show/use DepthB

show/use MinimumWallC

compute Actual Wall Remaining

AmountBelowMin becomes relevant and is saved. 

5. User-action triggers that should fire recalculation
From the current screen logic, recalculation already happens on C field change. The same pattern should also be applied to B field change, because both formulas depend on B and C. The existing C-field logic proves the intended event pattern: update the child record, recompute WallCalculation, then mark the parent dirty. 

Recommended trigger set
When DepthB changes

When MinimumWallC changes

Optionally when DRFType context changes by selecting a different parent row, the display label and visibility rules should refresh automatically from that selected parent record. 

6. Recommended normalized formula for U035
Since you said you need the dynamic formula for load changes and calculation changes logic, I would normalize it into one reusable expression keyed off the selected row’s parent DRFType.

Reusable calculation rule
If(
    varCurrentTarget.DRFType = "A",
    If(
        ValueB <> 0 && ValueC <> 0,
        (ValueB / ValueC) * 100,
        0
    ),
    ValueC - ValueB
)
For your actual child row patch, conceptually:

Patch(
    colChildResults,
    ThisItem,
    {
        DepthB: newB,
        MinimumWallC: newC,
        WallCalculation:
            If(
                varCurrentTarget.DRFType = "A",
                If(newC <> 0, (newB / newC) * 100, 0),
                newC - newB
            )
    }
);

Patch(
    colParentResults,
    LookUp(
        colParentResults,
        InspectionResultId = varCurrentTarget.InspectionResultId
    ),
    { IsDirty: true }
)
That is an inference from your current working direction because the old YAML snippet uses varDRF, but your current project direction is to use varCurrentTarget for the new modal flow. The calculation branch itself is supported; only the variable name shift is the project-specific adaptation. 

7. What changes across the three MOD PDFs
From the material retrieved here, I can confirm the three source PDFs are:

MOD 01

MOD 70

MOD 80 
 
 

What I could confirm from the current app sources is:

MOD selection changes which requirement set is loaded via nfModNumber. 

The DRF child schema and save logic do not vary by MOD in the current implementation. 

The DRF calculation branch varies by DRFType A/B, not by MOD. 
