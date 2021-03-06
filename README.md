Vital Signs Extraction System
------------------------------

Assessment of vital signs is an essential part of surveillance of critically ill patients to detect condition changes and clinical deterioration. While most modern electronic medical records allow for vitals to be recorded in a structured format, the frequency and quality of what is electronically stored may differ from how often these measures are actually recorded. We created a tool that extracts blood pressure, heart rate, temperature, respiratory rate, blood oxygen saturation, and pain level from nursing and other clinical notes recorded in the course of inpatient care to supplement structured vital sign data. 

If you use this system, please cite:

Patterson OV, Jones M, Yao Y, Viernes B, Alba PR, Iwashyna TJ, DuVall SL. Extraction of Vital Signs from Clinical Notes. Stud Health Technol Inform. 2015; 216:1035. Available from: http://www.ncbi.nlm.nih.gov/pubmed/26262334 




### Pipeline modules

- createNumericPipeline
   - NumericAnnotator
   - RegexAnnotator - GROOVY_CONFIG_FILE => IntegerNumber, DoubleNumber
	- AnnotationFilter -- TYPE_NUMERIC with REMOVE_CHILDREN=true
	- TimestampAnnotator -- RegexAnnotator => TYPE_TIMESTAMP
	- ExcludeNumberPattern -- AnnotationPatternAnnotator => TYPE_NUMEXCLUDE
	- AnnotationFilter - remove TYPE_INDICATOR
	- AnalyzeNumbersAE -- removeOverlappingAnnotations(NumericExclude, Numeric), number.setValue

- createTermAndIndicatorPipeline
	
	- UnitsAnnotator 
	- RegexAnnotator - GROOVY_CONFIG_FILE -> Unit with concept
	- TermAnnotator 
	- RegexAnnotator - GROOVY_CONFIG_FILE -> Bp_Term, Resp_Term, Hr_Term, ..., NotIt_Term
	- AnnotationFilter -- remove TYPE_TERM, TYPE_UNIT - REMOVE_CHILDREN=true
	- AnnotationFilter -- remove TYPE_TERM if overlapps with TYPE_UNIT
	- IndicatorPatternAnnotator 
	- AnnotationPatternAnnotator -> TYPE_INDICATOR
	- AnnotationFilter - remove TYPE_INDICATOR
	- TermExcludePatternAnnotator 
	- AnnotationPatternAnnotator -> TYPE_TERMEXCLUDE (start of an irrelevant section)
	
- createWindowsPipeline

	- WindowAnnotator - from TYPE_INDICATOR 20 tokens to the right -- HiPrecisionWindow
	- WindowAnnotator - from TYPE_INDICATOR 50 tokens to the right -- LowerPrecisionWindow
	- WindowAnnotator - from TYPE_TERMEXCLUDE 10 tokens to the right -- ExcludeAllWindow
	- AnnotationFilter - remove Numeric if overlaps with ExcludeAllWindow. REMOVE_CHILDREN=true

- createPatternsPipeline
	
	- RangePattern -- AnnotationPatternAnnotator => TYPE_RANGE
	- AdjustRangeAnnotator -- remove covered TYPE_RANGE, adjust span to include only the values, set value1 and value2
	- PotentialBpPattern -- AnnotationPatternAnnotator => TYPE_POTENTIAL_BP
	- ExcludeBpPattern -- AnnotationPatternAnnotator => TYPE_EX_POTENTIAL_BP
	- AnnotationFilter - remove TYPE_POTENTIAL_BP covered by TYPE_EX_POTENTIAL_BP
	- AdjustPotentialBpAE -- remove covered TYPE_POTENTIAL_BP, adjust span to include only the values, set value1 and value2
	- RelationPatternAnnotator -- AnnotationPatternAnnotator => TYPE_RELATION
	- RelationWithTimePatternAnnotator -- AnnotationPatternAnnotator => TYPE_RELATION_TIMESTAMP

- createVitalRulesPipeline
	- MarkNotItAE 
	- ExtractTemperatureAE 
	- ExtractSo2AE
	- ExtractBloodPressureAE
	- ExtractRespiratoryAE
	- ExtractHeightAE
	- ExtractWeightAE
	- ExtractPainAE
	- ExtractHeartRateAE
	- AnnotationFilter -- remove all covered annotatations of the same type PipelineVariables.valueTypes (all but BP)
	- AnnotationFilter -- remove all covered annotatations of the same type PipelineVariables.valueBPTypes
	- AnnotationFilter -- remove all general BP annotations if overlap with systolic BP or diastolic BP
	- FilterTimestampAE -- remove all time stamps that were not included in any of the Output_Value
