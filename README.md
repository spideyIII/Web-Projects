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

____________________________________________________________________________________________________________________________________________________________________________________
the PDFs support a text/topic-based filter that is more reliable than trying to force a table join. The stable match is not a hidden key between EGL and DRFs; it is the component/topic language repeated across the EGL and the DRF form titles, such as “torpedo tube,” “slide valve,” “breech door,” “muzzle door,” and similar equipment phrases. For a subtask like Torpedo Tubes, you can load all DRF titles in that mod containing “torpedo tube” and let the user pick. 
 

What the PDFs show
The EGL clearly repeats stable component names at the parent and subtask level:

1.A. TORPEDO TUBES

1.A.(1) Inspect interior and exterior surfaces of the torpedo tubes

1.A.(2) ... slide valve

1.A.(5) ... breech doors

1.A.(6) ... muzzle doors 
 
 

The DRF pages also use stable topic titles, for example:

TORPEDO TUBE SLIDE VALVE DEFECTS

TORPEDO TUBE DEFECTS (INCLUDING CONNECTIONS 1 INCH AND GREATER) 
 

That means there is a defensible filter path:

filter by Mod

derive a search term from the EGL parent/subtask text

filter DRF titles by that term

Best match key to use
The safest practical match key is a normalized topic keyword, not a row join.

For example:

TORPEDO TUBES parent → search DRF titles for "torpedo tube"

slide valve subtask → search DRF titles for "slide valve"

breech doors → search for "breech door"

muzzle doors → search for "muzzle door"

This is supported by the wording in the PDFs because the same equipment/component language appears in both the EGL and the DRF sections. 
 

For your specific example
If the selected subtask is under Torpedo Tubes, then yes — show all DRF titles for that mod containing “torpedo tube”.

That is especially supported because the PDFs explicitly include DRF titles like:

TORPEDO TUBE SLIDE VALVE DEFECTS

TORPEDO TUBE DEFECTS (INCLUDING CONNECTIONS 1 INCH AND GREATER) 
 

Recommended logic
Use a small keyword map from the selected EGL row.

Suggested priority
If subtask text contains slide valve → filter DRFs by slide valve

Else if contains breech door → filter by breech door

Else if contains muzzle door → filter by muzzle door

Else if contains flood and drain or connections → filter by torpedo tube

Else if parent title contains torpedo tube → filter by torpedo tube

Else fall back to all DRFs for the mod

_____________________________________________________________________________________________________________________________________________________________________________
Mod 01 — General Notes
Tools, Parts, Material & Test Equipment: None. 

The requirements within this URO MRC shall not be changed unless specifically approved by NAVSEA (05U7). 

A SUBMEPP Product Feedback Form is provided at the end of this URO MRC for the user to report configuration changes and identify any user comments associated with this URO MRC. 

The intent of this inspection is to examine to the maximum extent practical the internal and external surfaces of components listed in the Equipment Guide List (EGL) to identify as arrived degradation or damage. 

100% of the insulation must be removed from the Trash Disposal Unit/Ejector Barrel to accomplish this URO MRC. Reinstallation of the insulation is not required. 

Screwed or bolted-in gratings do not constitute an inaccessible condition for inspection, specifically the gratings covering the slide valves on torpedo tubes. 

Any defect below minimum drawing dimensions for certain areas of the slide valve wall thickness is unacceptable based on SUBSAFE Design Review; refer to Step 2.C. for guidance. 

Inform NAVSEA (05U7), by faxed letter or message, of any major repairs planned as a result of conducting inspections. Major repairs include structural effects to pressure-containing portions through welding, brazing, or similar fabrication, and installation of new pressure-boundary parts/components not previously strength tested. 

If delay of ship is involved or if deferring specific repairs is desired for other valid reasons, forward information to Type Commander for detailed guidance. Identify location, extent, and type of defect. 

Applicable hull and backup valves associated with listed components shall be inspected in accordance with URO MRC 008. Hull welds and hull inserts and/or liners shall be inspected in accordance with URO MRCs 001 and/or 003, as applicable. TDU muzzle valve and signal ejector (or equivalent) hull valves do not require inspection by this URO MRC. 

A Major Departure from Specification is required to be submitted to NAVSEA for approval for any component, other than Torpedo Tube Slide Valves, found in Category B or C condition and remaining in service. 

Mod 70 — General Notes
Tools, Parts, Material & Test Equipment: Depth gage, inside/outside micrometers, inside/outside calipers, flashlight. 

The requirements within this URO MRC shall not be changed unless specifically approved by NAVSEA (05U7). 

A SUBMEPP Product Feedback Form is provided at the end of this URO MRC for the user to report configuration changes and identify any user comments associated with this URO MRC. 

The intent of this inspection is to examine to the maximum extent practical the internal and external surfaces of components listed in the Equipment Guide List (EGL) to identify deterioration or damage. 

Screwed or bolted-in gratings do not constitute an inaccessible condition for inspection, specifically the gratings covering the slide valves on torpedo tubes. 

Inform NAVSEA (05U7), by faxed letter or message, of any major repairs planned as a result of conducting inspections. Major repairs include structural effects to pressure-containing portions through welding, brazing, or similar fabrication, and installation of new pressure-boundary parts/components not previously strength tested. 

If delay of ship is involved or if deferring specific repairs is desired for other valid reasons, forward information to Type Commander for detailed guidance. Identify location, extent, and type of defect. 

Applicable hull and backup valves associated with listed components shall be inspected in accordance with criteria of URO MRC 008. Hull welds, hull inserts and/or liners, and torpedo tube lands and their associated non-ferrous welds shall be inspected in accordance with URO MRCs 001 and/or 003 as applicable. TDU muzzle valve and ICL (or equivalent) hull valves do not require inspection by this URO MRC. 

A Major Departure from Specification is required to be submitted to NAVSEA for approval for any component, other than Torpedo Tube Slide Valves, found in Category B or C condition and remaining in service. 

Any defect greater than 0.030" below minimum drawing dimensions for slide valve wall thickness is unacceptable based on SUBSAFE Design Review. Refer to Procedure Section Step 2.C. for guidance. 

Mod 80 — General Notes
The retrieved Mod 80 text shows 4 General Notes. 

The requirements within this URO MRC shall not be changed unless specifically approved by NAVSEA (05U7). 

A SUBMEPP Product Feedback Form is provided at the end of this URO MRC for the user to report configuration changes and identify any user comments associated with this URO MRC. 

Inform NAVSEA (05U7), by letter or transmittal, of any major repairs planned as a result of conducting inspections. Major repairs include structural effects to pressure-containing portions through welding, brazing, or similar fabrication, and installation of new pressure-boundary parts/components not previously strength tested. 

Applicable hull and backup valves associated with listed components shall be inspected in accordance with criteria of URO MRC 008. Hull welds, hull inserts and/or liners, and torpedo tube lands and their associated non-ferrous welds shall be inspected in accordance with URO MRCs 001 and/or 003 as applicable. TDU and IAL muzzle valves do not require inspection. 

The DRF calculation branch varies by DRFType A/B, not by MOD. 
<img width="1624" height="1132" alt="image" src="https://github.com/user-attachments/assets/3afcad8c-d5ab-4e6d-8de8-fa74950396d5" />
<img width="1286" height="676" alt="image" src="https://github.com/user-attachments/assets/1cad7048-c484-40a0-9bcc-fc9c197ba856" />
<img width="1188" height="485" alt="image" src="https://github.com/user-attachments/assets/f6f82062-d71d-4971-bc79-64eeff16615f" />

LookUp(colRefImages, Text('PK - Impact URO Reference Image') = Coalesce(_refImageKey, Text(First(colRefImages).'PK - Impact URO Reference Image'))).'Reference Image'.Full


