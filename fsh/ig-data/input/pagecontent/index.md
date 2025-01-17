<h3 id="drug-formulary">DaVinci Payer Data Exchange US Drug Formulary</h3>
<p>This project defines a FHIR interface to a health insurer's drug formulary information for patients/consumers. 
A drug formulary is a list of brand-name and generic prescription drugs a health insurer 
    agrees to pay for, at least partially, as part of health insurance coverage.
    Drug formularies are developed based on the efficacy, safety and cost of drugs. 
   The primary <a href="#use-cases">use cases</a> for this FHIR interface enable consumers/members/patients to 
   understand the costs and alternatives for drugs that have been prescribed, and to compare their drug
  costs across different insurance plans.</p>
<p>
A key architectural issue that is beyond the scope of this implementation guide is how a user finds the FHIR endpoint for a particular formulary.    This implementation guide assumes that the FHIR endpoint is known to the user. </p>
  <h4 id="table-of-contents">Table Of Contents</h4>
<ul>
<li><a href="#introduction">Introduction</a></li>
<li><a href="#disclaimers-assumptions">Disclaimers and Assumptions</a></li>
<li><a href="#formulary-structure">Formulary Structure</a></li>
<li><a href="#expected-users">Expected Users</a></li>
<li><a href="#use-cases">Use Cases</a></li>
<li><a href="#privacy-considerations">Privacy Considerations</a></li>
<li><a href="#anticipated-client-queries">Anticipated Client Queries</a></li>
<li><a href="#open-issues">Additional Background and Open Issues</a></li>
<li><a href="#credits">Credits</a></li>
<li><a href="#authors">Authors</a></li>
</ul>

  <h4 id="introduction">Introduction</h4>
  <p>This implementation guide (IG) introduces  2 FHIR profiles, along with associated extensions, search parameters, and value sets.</p>

  <ul>
      <li><b><a href="StructureDefinition-usdf-CoveragePlan.html">CoveragePlan</a></b>: The CoveragePlan profile of the FHIR R4 <a href="http://hl7.org/fhir/R4/list.html">List</a> resource provides 
        links to information about the plan and formulary, contact information, a description of the drugTiers and associated cost sharing models of the plan, and a list of FormularyDrugs. </li>    
      <li><b><a href="StructureDefinition-usdf-FormularyDrug.html">FormularyDrug</a></b>: The FormularyDrug profile of the FHIR R4 <a href="http://hl7.org/fhir/medicationknowledge.html">MedicationKnowledge</a> resource provides plan-specific
      information about a prescribable drug identified by an RxNORM identifier.  Cost sharing for the drug are described by reference to a drug tier defined as part of the coverage plan.  Extensions to the MedicationKnowledge resource support
    important search use cases. Due to the immaturity of the MedicationKnowledge resource, it is expected that it will undergo changes, and those changes may require evolution of the FormularyDrug profile.
  </li>
  </ul>
  Two searchParameters have also been defined to facilitate the anticipated use cases.  See the <a href="#anticipated-client-queries">Anticipated Client Queries</a> section for a description of how to query the CoveragePlan and FormularyDrug profiles
  in support of the anticipated use cases.
  <ul>
      <li><b><a href="SearchParameter-DrugPlan.html">DrugPlan</a></b>:  Makes the DrugPlan identifier of each FormularyDrug accessible for query   </li>
      <li><b><a href="SearchParameter-DrugTier.html">DrugTier</a></b>: Makes the DrugTier identifier of each FormularyDrug accessible for query    </li>
    </ul>
  
  <h4 id="disclaimers-assumptions">Disclaimers and Assumptions</h4>
<ul>
 <li><b>Drug Formulary includes Plan-Level Data Only</b>:  The intent of this implementation guide is to make the plan-level information regarding formulary content and cost-sharing available through a standard interface for third party applications.  Most users will access this data through a third party application.   That application should clearly communicate to the user that the cost-sharing information in the formulary may not tell them precisely what they will pay at the pharmacy, and might not reflect their drug benefit.   There is an ongoing effort by Carin/NCPDP to develop a Real Time Pharmacy Benefit Check (RTPBC) implementation guide.   Future ballots of this implementation guide and the RTPBC implementation guide should be harmonized.
</li>
<li><b>The MedicationKnowledge Resource is immature</b>:  When designing this IG, we initially planned to profile Medication.  Based on a strong recommendation from the PharmaWG we shifted to profiling Medicationknowledge.   The PharmaWG felt that MedicationKnowledge was a better choice, even with its low maturity, since it is more suitable as an artifact and already included some of fields that would be required as extensions to medication (e.g., classification). MedicationKnowledge and FormularyDrug will co-evolve moving forward.
</li>
<li><b>The formulary endpoint is known to the client</b>: This IG assumes that the formulary endpoint is known to the client.  There is an overarching system architecture issue that is critical to resolve -- how does the client discover the FHIR endpoint of interest.   For the purposes of this IG, we consider that problem out of scope.
</li>
</ul>
<h4 id="formulary-structure">Formulary Structure</h4>
  <p>Formularies in the United States are normally published by health insurers on an annual basis, with minor updates during the year. It is critical that health insurers update their published Formularies following these minor updates. 
  <p>Insurers regularly administer multiple health insurance and drug coverage plans and each of those plans may have its own formulary.</p>
  <p>
Each formulary contains a list of drugs.
  Drugs are placed into <strong>tiers</strong> that largely determine the cost to the consumer/patient. 
  The number and purpose of drug tiers varies across payers.  
  Each tier has an associated cost-sharing model that includes deductibles and/or coinsurance components for drugs in the tier
  when purchased through various pharmacy types.</p>
  <p>In addition to the drug tier, drugs may also list requirements on the patient (e.g., age or gender) or limitations on 
      prescription (e.g., prior authorization). </p>
  <p>
  This Implementation Guide closely follows the formulary information model of the 
  <a href="https://github.com/CMSgov/QHP-provider-formulary-APIs">formularies for Qualified Health Plans (QHPs) on the federal health insurance marketplace for healthcare.gov</a>.
  Publishing formularies in the QHP format should be familiar to many payers.  Drugs are specified by RxNorm codes of prescribable drugs, as 
  constrained by the <a href="https://build.fhir.org/ig/HL7/US-Core-R4/ValueSet-us-core-medication-codes.html">US Core Medication Codes value set</a>.
  The QHP data model mandates specific value sets for some data types (e.g., types of copayments), but leaves value sets for other 
  data types at the discretion of the payer (e.g., drug tier identifiers, pharmacy types). and does not include data that is fairly standard across formularies (drug classifications, alternative drugs).
 The following object model shows the relationships between the resources in the current IG. The only areas where this Implementation Guide extends beyond the QHP formulary information model is the addition of DrugClass and DrugAlternatives to FormularyDrug, as highlighted in the figure.
  </p>
 <p>
</p>

    <img style="width:100%;height:auto;" src="formularyQHP.jpg" />
  <p>
  A FormularyDrug represents the availability of a drug with a specific RxNorm code within the tier structure and prescribing constraints of a specific CoveragePlan.   
  If a FHIR endpoint provides data on multiple CoveragePlans, querying for FormularyDrugs by their RxNorm code would return multiple entries.   Each of these FormularyDrugs
  could associate the drug to a distinct DrugTier in the associated CoveragePlan, with plan-specific prescribing constraints.   
  The CoveragePlan PlanID field and the FormularyDrug PlanID extension field associate a FormularyDrug with a CoveragePlan.
  </p>
  <h4 id="expected-users">Expected Users</h4>
  <p>
  This Implementation Guide is intended for insurers within the United States. 
  Currently, many insurers make their formularies available to patients using PDFs or drug search forms through their websites. 
  Providing formularies using FHIR may allow patients to more easily comparison-shop between plans and could help insurers educate 
  consumers about the differences between various drug tiers/classes.</p>
  <p>&nbsp;</p>
  
<h4 id="use-cases">Use Cases</h4>
<h5 id="actors">Actors</h5>
<ul>
<li>Member</li> A person to whom health care coverage has been extended by the policyholder (generally their employer) or any of their covered family members. Sometimes referred to as the insured or insured person.  The cost of medications for a member is a function of the drug coverage under their health insurance plan, their benefits based on their already satisified deductibles, and the pharmacy where their prescriptions are filled.
<li>Consumer</li> A person who may or may not be a member, who wishes to explore the formulary content and plan-level co-insurance.
<li>Health Plan</li> A provider of drug coverage who publishes their formulary content and plan-level co-insurance information through the Drug Formulary FHIR API.
</ul>
<h5>Med Copays under Health Plan</h5>
<p>
This use case allows a member to determine the plan level estimated costs of each of their medications under the drug coverage of their current health plan.  
The mobile application retrieves the member's medication list from an electronic health record system where the member's patient data is stored. This security and privacy of a patient's access to their health information is beyond the scope of this Implementation Guide.  The member could also independently maintain their medication list in the mobile application or elsewhere.
The mobile application queries the formulary service for cost information about the drugs that the member takes and provides 
the the plan level estimated cost for each medication under the member's current health plan.
</p>
<p>
Note that for this use case the coverage plan could provide authenticated or open access to the plan formulary, and the privacy of the member's data is protected.
</p>
<p>
<img style="width:100%;height:auto;" src="Slide1.jpg" />
<p>&nbsp;</p>
<h5>Shopping for Health Plans</h5>
<p>
This use case allows a consumer to compare the drug coverage of several different health plans and determine which plan has the lowest plan level estimated cost, 
personalized to the consumers's set of medications.  
The mobile application retrieves the consumer's medication list from an electronic health record system where the consumer's patient data is stored. This security and privacy of a patient's access to their health information is beyond the scope of this Implementation Guide.  The consumer could also independently maintain their medication list in the mobile application or elsewhere.   The mobile application identifies the relevant formulary endpoint through means that are beyond the scope of this implementation guide (see <a href="#disclaimers-assumptions">Disclaimers and Assumptions</a>).
For each payer, the mobile application queries the payer's formulary service to retrieve the list of health plans provided by that payer. Then, for each plan,the mobile application queries the formulary service to retrieve the plan-level estimated costs specific to the consumer's medication list. 
<p>
Access to the formulary service should not require authentication, and the server should not maintain any records that could associate the consumer with the medication list that was queried.</p>
<p>&nbsp;</p>

<img style="width:100%;height:auto;" src="Slide2.jpg" />
<p>&nbsp;</p>
<h4 id="privacy-considerations">Privacy Considerations</h4>
<p>
Access to the formulary service should not require authentication, and the server should not maintain any records that could associate the consumer with the medication list that was queried.</p>
<p>
A conformant payer formulary service SHALL NOT require a formulary mobile application to send consumer identifying information in order to query for the list of health plans provided by that payer and the medication costs for each plan, specific to the consumer's set of medications.</p>
<p>
A formulary mobile application SHALL NOT send consumer identifiable information when querying a formulary service. </p>
<h4 id="anticipated-client-queries">Anticipated Client Queries</h4>
<h5 id="Find-all-CoveragePlans"> Find All CoveragePlans </h5> 
<pre><code>
    GET [base]/List?_profile=http://hl7.org/fhir/us/Davinci-drug-formulary/StructureDefinition/usdf-CoveragePlan
</code>
</pre>
<p>
For each CoveragePlan, the PlanID is mapped to the List.identifier field.   
The value of List.identifier is the most general way to query the FormularyDrugs that are 
part of a specific plan.
</p>
<h5 id="Find-a-CoveragePlan-by-planid"> Find CoveragePlan by its PlanID </h5> 
<p>
To find the CoveragePlan for a plan with id 'myPlanID':
<pre><code>
  GET [base]/List?_profile=http://hl7.org/fhir/us/Davinci-drug-formulary/StructureDefinition/usdf-CoveragePlan&amp;identifer=myPlanID 
</code>
</pre>
<h5 id="Find-all-FormularyDrugs-in-a-CoveragePlan">Find All FormularyDrugs in a CoveragePlan</h5>
<p>
To find all FormularyDrugs in a CoveragePlan for a plan with id 'myPlanID':
</p>
<pre><code>
    GET [base]/MedicationKnowledge?_profile=http://hl7.org/fhir/us/Davinci-drug-formulary/StructureDefinition/usdf-FormularyDrug&amp;DrugPlan=myPlanID 
</code></pre>
<p>
Alternatively, these FormularyDrugs are also in the array of entries that is part of the List.
</p>
<h5 id="Find-all-FormularyDrugs-in-a-CoveragePlan-DrugTier">Find All FormularyDrugs in a Specific Tier of CoveragePlan</h5>
<p>
To find all FormularyDrugs in the GENERIC tier of plan myPlanID:
</p>
<pre><code>
    GET [base]/MedicationKnowledge?_profile=http://hl7.org/fhir/us/Davinci-drug-formulary/StructureDefinition/usdf-FormularyDrug&amp;DrugPlan=myPlanID&amp;DrugTier=GENERIC
</code></pre>
<h5 id="Find-a-FormularyDrugs-by-code-in-a-CoveragePlan">Find A FormularyDrug by code in a CoveragePlan</h5>
<p>
To find a FormularyDrug by its RxNorm code within a CoveragePlan:
</p>
<pre><code>
    GET [base]/MedicationKnowledge?_profile=http://hl7.org/fhir/us/Davinci-drug-formulary/StructureDefinition/usdf-FormularyDrug&amp;DrugPlan=myPlanID&amp;code=myCode
</code></pre>
<h5 id="Find-a-FormularyDrugs-by-code-across-all-coverage-plans">Find A FormularyDrug by code across all CoveragePlans</h5>
<p>
To find a FormularyDrug by its RxNorm code within all CoveragePlans:
</p>

<pre><code>
    GET [base]/MedicationKnowledge?_profile=http://hl7.org/fhir/us/Davinci-drug-formulary/StructureDefinition/usdf-FormularyDrug&amp;code=myCode
</code></pre>
<h3 id="open-issues">Additional Background and Open Issues</h3>
  
  <h4 id="presenting-alternative-medications">Presenting Drug Alternatives</h4>
  <p>There may be brand or generic alternatives to a particular drug in the formulary.  The QHP formulary information model, does not include drug alternatives. 
  The current Implementation Guide provides for each FormularyDrug to include an array of references to other FormularyDrugs within the same CoveragePlan.
  There may be better ways to represent equivalence classes among FormularyDrugs using the DrugClass.
   </p>
  <h4 id="representing-drug-tiers">Representing Drug Tiers</h4>
  <p>
  Drug tiers are not standardized.   The current Implementation Guide provides a defined, but extensible value set for tier identifiers based on the example list in the QHP formulary
  specification.   A move towards standardization might make this data more useful for clients of the interface.
  </p>
  <h4 id="representing-drug-classifications">Representing Drug Classifications</h4>
  <p>
 Within a consumer-facing drug formulary the primary use of drug classification is to enable hierarchical browsing of the formulary contents from a
 therapeutic disease area (e.g., hypertension) or pharmacologic action (e.g., beta blocker) perspective.  An empirical review of web/PDF-based drug formularies found
 variety of different hierarchies being used to present the formulary to consumers. The current IG suggests the utility of using the FormularyDrug.medicineClassification field
 to provide drug classification information, but does not specify a particular vocabulary.   This might be a fruitful area for subsequent standardization.
  </p>
  <h4 id="representing-pharmacy-types">Representing Pharmacy Types</h4>
  <p>
  Pharmacy types are not standardized.   The current Implementation Guide provides a defined value set for tier identifiers based on the example list in the QHP formulary
  specification which mixes channels (retail and mail order) with quantity prescribed (1 month, 3 month, etc).   
  A move towards standardization might make this data more useful for clients of the interface.
    </p>
  <h4 id="provision-of-formulary-ids">Provision of Formulary IDs and Availability of Directory</h4>
  <p>There is no single, authoritative indentifier that can be associated with a formulary (e.g., like NPI numbers identify providers in the United States). 
    How can unique formulary IDs be provisioned such that they can be implemented consistently by all payers and referenced by other entities (e.g., health plans)?
    The NCPDP Formulary and Benefits eRx implementation guide requires an identifier for each formulary.  Perhaps that can be leveraged.   
   </p>

   <h3 id="credits">Credits</h3>
   This IG was developed by the MITRE Corporation using the free, open source,  <a href="http://standardhealthrecord.org/cimpl-doc/">Clinical Information Modeling and Profiling Language (CIMPL)</a> tool chain.
   The CIMPL inputs used to generate this IG with <a href="https://github.com/standardhealth/shr-tools/releases/tag/shr-cli%406.10.4">shr-cli-6.10.4</a> can be found <a href="https://github.com/HL7-DaVinci/pdex-formulary-cimpl/tree/STU1Release">here</a>. 
  
  <h3 id="authors">Authors</h3>
  <table>
      <thead>
      <tr>
      <th>Name</th>
      <th>Email</th>
      </tr>
      </thead>
      <tbody>
      <tr>
      <td>Saul A. Kravitz</td>
      <td><a href="mailto:saul@mitre.org">saul@mitre.org</a></td>
      </tr>
      <tr>
      <td>May Terry</td>
      <td><a href="mailto:mayt@mitre.org">mayt@mitre.org</a></td>
      </tr>
      <tr>
      <td>Chris Klesge</td>
     <td><a href="mailto:cklesges@mitre.org">cklesges@mitre.org</a></td>
      </tr> 
    </tbody>
  </table>
