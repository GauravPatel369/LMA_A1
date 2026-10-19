# Answer 1 (baseline) vs Answer 2 (re-ranked)

## Q1 - negation / jargon gap

**Query:** Can a bank refuse my loan application using only an algorithm with no human involved?

**Gold:** Article 22 / Recital 71 - rank #4 -> #1

**Baseline context:** ['Recital 18', 'Recital 43', 'Recital 28']

**Answer 1:**

> The provided context does not contain the answer to this question. The excerpts provided (Recitals 18, 43, and 28) discuss the scope of the Regulation regarding household activities, the conditions for valid consent, and the use of pseudonymization, but they do not address automated decision-making, algorithmic lending, or the right to human intervention in credit decisions.

**Re-ranked context:** ['Recital 71', 'Recital 18', 'Recital 80']

**Answer 2:**

> Based on the provided context, a bank generally **cannot** refuse a loan application using only an algorithm with no human involved, unless specific exceptions apply.

**Recital 71** states that a data subject has the right "not to be subject to a decision... which is based solely on automated processing and which produces legal effects concerning him or her or similarly significantly affects him or her, such as automatic refusal of an online credit application... without any human intervention."

However, **Recital 71** also notes that such decision-making is allowed if it is:
1.  "expressly authorised by Union or Member State law... including for fraud and tax-evasion monitoring and prevention purposes... and to ensure the security and reliability of a service provided by the controller";
2.  "necessary for the entering or performance of a contract between the data subject and a controller"; or
3.  Based on the data subject's explicit consent (the text cuts off at "or when the data subject", implying this standard GDPR exception, but strictly relying on the provided text, the first two conditions are explicitly listed).

Therefore, a purely automated refusal is prohibited unless it falls under the specific legal authorizations or contractual necessities mentioned in **Recital 71**.

---

## Q2 - keyword trap / distractor

**Query:** When must we tell the affected individuals themselves about a breach?

**Gold:** Article 34 - rank #5 -> #2

**Baseline context:** ['Recital 86', 'Recital 85', 'Article 33']

**Answer 1:**

> You must communicate the personal data breach to the data subject **without undue delay** when the breach is **likely to result in a high risk** to the rights and freedoms of the natural person.

The communication should be made **as soon as reasonably feasible** and in close cooperation with the supervisory authority.

**Source:** Recital 86

**Re-ranked context:** ['Recital 85', 'Article 34', 'Recital 86']

**Answer 2:**

> You must communicate the personal data breach to the data subject **without undue delay** when the breach is **likely to result in a high risk to the rights and freedoms of natural persons**.

This obligation is subject to exceptions where communication is not required if:
1.  The controller has implemented appropriate technical and organisational protection measures (such as encryption) that render the personal data unintelligible to unauthorised persons;
2.  The controller has taken subsequent measures ensuring that the high risk is no longer likely to materialise; or
3.  Communication would involve disproportionate effort (in which case, a public communication or similar measure must be used instead).

**Sources:**
*   **Article 34(1)**: "When the personal data breach is likely to result in a high risk to the rights and freedoms of natural persons, the controller shall communicate the personal data breach to the data subject without undue delay."
*   **Article 34(3)**: Lists the conditions under which communication to the data subject is not required.
*   **Recital 86**: Confirms the controller should communicate "without undue delay" where the breach is "likely to result in a high risk to the rights and freedoms of the natural person."

---

## Q3 - buried clause / role limit

**Query:** If two companies decide together how data is used, who is responsible?

**Gold:** Article 26 - rank #8 -> #1

**Baseline context:** ['Recital 40', 'Recital 74', 'Recital 79']

**Answer 1:**

> Based on the provided context, if two companies decide together how data is used (determining the purposes and means of the processing jointly), the responsibility is allocated to them as **joint controllers**.

**Recital 79** states that the Regulation requires "a clear allocation of the responsibilities... including where a controller determines the purposes and means of the processing jointly with other controllers."

Additionally, **Recital 74** establishes that "The responsibility and liability of the controller for any processing of personal data carried out by the controller or on the controller's behalf should be established," and the controller is obliged to implement appropriate measures and demonstrate compliance.

**Re-ranked context:** ['Article 26', 'Recital 79', 'Article 47']

**Answer 2:**

> Based on the provided context, if two companies decide together how data is used (i.e., they jointly determine the purposes and means of processing), they are **joint controllers**.

They are responsible for determining their respective responsibilities for compliance with the obligations under the Regulation in a transparent manner. This includes responsibilities regarding the exercising of data subject rights and providing information referred to in Articles 13 and 14. They must establish an arrangement between them to reflect their respective roles and relationships vis-à-vis the data subjects, unless their responsibilities are already determined by Union or Member State law.

**Source:** Article 26 - Joint controllers (Paragraphs 1 and 2).

---

## Q4 - constraint / buried clause

**Query:** Should privacy be built into a product while it is being designed?

**Gold:** Article 25 - rank #4 -> #7

**Baseline context:** ['Recital 78', 'Recital 63', 'Recital 7']

**Answer 1:**

> Yes, privacy should be built into a product while it is being designed. According to **Recital 78**, producers of products, services, and applications should be encouraged to "take into account the right to data protection when developing and designing such products, services and applications." This recital explicitly references the principles of "data protection by design and data protection by default" as measures that controllers should implement to demonstrate compliance.

**Re-ranked context:** ['Recital 78', 'Recital 4', 'Recital 78']

**Answer 2:**

> Yes, privacy should be built into a product while it is being designed. According to **Recital 78**, producers of products, services, and applications should be encouraged to "take into account the right to data protection when developing and designing such products, services and applications." This aligns with the principles of "data protection by design and data protection by default," which require that appropriate technical and organisational measures be taken to ensure compliance with the Regulation.

---

## Q5 - role limit / contract

**Query:** What has to be written in the contract with a company that handles data for us?

**Gold:** Article 28 - rank #6 -> #7

**Baseline context:** ['Article 13', 'Recital 155', 'Recital 81']

**Answer 1:**

> Based on the provided context, the specific content that must be written in the contract is not fully detailed, but the context indicates that the controller and processor may choose to use an **individual contract** or **standard contractual clauses**.

These clauses are adopted either directly by the Commission or by a supervisory authority in accordance with the consistency mechanism and then adopted by the Commission.

**Source:** Recital 81

**Re-ranked context:** ['Article 13', 'Recital 40', 'Recital 155']

**Answer 2:**

> The provided context does not contain the answer to this question.

The context includes Article 13 (regarding information to be provided to data subjects), Recital 40 (regarding lawful bases for processing), and Recital 155 (regarding employee data). It does not contain the text of Article 28 (which typically governs the requirements for contracts between a controller and a processor) or any other provision specifying the mandatory clauses for a contract with a data handler/processor.

---

## Q6 - regression probe

**Query:** How long are we allowed to keep customer records?

**Gold:** Article 5 - rank #1 -> #4

**Baseline context:** ['Article 5', 'Recital 39', 'Recital 171']

**Answer 1:**

> Based on the provided context, there is no specific fixed time limit (e.g., "5 years" or "10 years") stated for keeping customer records. Instead, the context provides the following principles:

1.  **Storage Limitation:** Personal data must be kept in a form which permits identification of data subjects for **no longer than is necessary** for the purposes for which the personal data are processed (**Article 5(1)(e)**).
2.  **Strict Minimum:** The period for which personal data are stored should be limited to a **strict minimum** (**Recital 39**).
3.  **Controller Responsibility:** The controller should establish **time limits** for erasure or for a periodic review to ensure data are not kept longer than necessary (**Recital 39**).
4.  **Exceptions:** Data may be stored for longer periods if processed solely for archiving purposes in the public interest, scientific or historical research purposes, or statistical purposes, subject to appropriate safeguards (**Article 5(1)(e)**).

Therefore, the allowed duration is determined by the specific purpose of the processing and must be the minimum necessary time, as defined by the controller.

**Re-ranked context:** ['Recital 137', 'Recital 39', 'Article 12']

**Answer 2:**

> Based on the provided context, there is no specific fixed duration (e.g., "5 years" or "10 years") stated for how long customer records may be kept.

However, **Recital 39** states that "the period for which the personal data are stored is limited to a strict minimum" and that "time limits should be established by the controller for erasure or for a periodic review" to ensure data are not kept longer than necessary.

Therefore, the allowed duration is not a set number of days or years, but rather a period that must be limited to a strict minimum and defined by the controller through established time limits for erasure or review.

---

## Q7 - recall ceiling (>10)

**Query:** What security measures protect against hackers stealing our database?

**Gold:** Article 32 - rank not in top-10 -> not in top-10

**Baseline context:** ['Recital 83', 'Recital 49', 'Recital 64']

**Answer 1:**

> Based on the provided context, the following security measures and considerations are mentioned to protect against risks such as unauthorized access (which includes hacking):

*   **Encryption:** Recital 83 explicitly lists "encryption" as a measure to mitigate risks inherent in processing.
*   **Preventing unauthorized access:** Recital 49 states that processing is necessary for ensuring network and information security, which includes "preventing unauthorised access to electronic communications networks."
*   **General risk mitigation:** Recital 83 states that controllers or processors should "evaluate the risks inherent in the processing and implement measures to mitigate those risks." These measures must ensure an appropriate level of security, including confidentiality, taking into account the state of the art and costs.

**Relied upon:** Recital 83, Recital 49.

**Re-ranked context:** ['Recital 83', 'Recital 88', 'Recital 28']

**Answer 2:**

> Based on the provided context, the specific security measures mentioned to mitigate risks (such as those inherent in processing that could lead to unauthorized access) are **encryption** and **appropriate technical protection measures**.

*   **Recital 83** states that controllers or processors should implement measures to mitigate risks, "such as **encryption**," to ensure an appropriate level of security, including confidentiality.
*   **Recital 88** refers to personal data being "protected by **appropriate technical protection measures**," which effectively limit the likelihood of misuse.

The context does not list other specific technical controls (such as firewalls or multi-factor authentication) beyond these examples.

---

## Q8 - recall ceiling (>10)

**Query:** Must we obey a foreign court order demanding European personal data?

**Gold:** Article 48 - rank not in top-10 -> not in top-10

**Baseline context:** ['Recital 4', 'Recital 50', 'Article 13']

**Answer 1:**

> The provided context does not contain the answer to this question. The excerpts provided (Recital 4, Recital 50, and Article 13) discuss the balance of fundamental rights, conditions for further processing, and information obligations to data subjects, but they do not address the legal requirements for complying with foreign court orders or international data transfer requests.

**Re-ranked context:** ['Recital 12', 'Recital 105', 'Recital 31']

**Answer 2:**

> The provided context does not contain the answer to this question.

The context includes:
*   **Recital 12**: Discusses the mandate of the European Parliament and Council regarding data protection rules and free movement.
*   **Recital 105**: Discusses the Commission's assessment of third-country protection levels, specifically mentioning the Council of Europe Convention of 28 January 1981.
*   **Recital 31**: Discusses public authorities (such as tax and customs authorities) receiving data for official missions and states they should not be regarded as recipients under specific conditions.

None of these excerpts address the specific legal obligation or procedure for complying with a **foreign court order** demanding European personal data.

---

## Q9 - exception / corner case

**Query:** Can a hospital process a patient's medical records in order to provide treatment?

**Gold:** Article 9 - rank #3 -> #2

**Baseline context:** ['Recital 63', 'Recital 69', 'Article 9']

**Answer 1:**

> Yes, a hospital can process a patient's medical records in order to provide treatment.

According to **Article 9 - Processing of special categories of personal data**, paragraph (h), processing is permitted when it is "necessary for the purposes of ... the provision of health or social care or treatment ... on the basis of Union or Member State law or pursuant to contract with a health professional and subject to the conditions and safeguards referred to in paragraph 3."

**Re-ranked context:** ['Recital 63', 'Article 9', 'Recital 91']

**Answer 2:**

> Yes, a hospital can process a patient's medical records in order to provide treatment.

According to **Article 9**, processing of special categories of personal data is permitted when it is "necessary for the purposes of ... medical diagnosis, the provision of health or social care or treatment or the management of health or social care systems and services on the basis of Union or Member State law or pursuant to contract with a health professional and subject to the conditions and safeguards referred to in paragraph 3."

---

## Q10 - exception / corner case

**Query:** If someone asks us to delete their data, when can we refuse?

**Gold:** Article 17 / Recital 65 - rank #1 -> #1

**Baseline context:** ['Recital 65', 'Recital 81', 'Article 17']

**Answer 1:**

> Based on the provided context, the text does not explicitly list specific grounds for **refusing** a request for erasure. Instead, it defines the grounds under which a controller **must** erase data (Article 17(1)) and notes that further retention may be lawful in specific circumstances (Recital 65).

However, the context implies that a controller is not obligated to erase data if the conditions in Article 17(1) are not met, and specifically mentions that retention is lawful in the following scenarios:

1.  **Freedom of Expression and Information**: Retention is lawful where it is necessary for exercising the right of freedom of expression and information (Recital 65).
2.  **Legal Obligation**: Retention is lawful where it is necessary for compliance with a legal obligation (Recital 65).
3.  **Processor Storage Requirements**: A processor may retain data if there is a requirement to store the personal data under Union or Member State law to which the processor is subject (Recital 81).

Additionally, Article 17(1)(c) states that the right to erasure applies when a data subject objects to processing **and** there are no overriding legitimate grounds for the processing. This implies that if there **are** overriding legitimate grounds for the processing, the controller may not be required to erase the data in response to that specific objection.

**Sources:**
*   Recital 65
*   Recital 81
*   Article 17(1)(c)

**Re-ranked context:** ['Recital 65', 'Article 17', 'Article 7']

**Answer 2:**

> Based on the provided context, the text does not explicitly list specific scenarios where a controller can **refuse** a request for erasure. Instead, it defines the specific grounds where the controller **must** erase the data (Article 17(1)) and notes that retention may remain lawful in certain circumstances (Recital 65).

However, by analyzing the conditions under which erasure is **mandatory** in Article 17(1), one can infer that a controller might not be obligated to erase data if none of the following grounds apply:
*   The data is still necessary for the original purposes.
*   The data subject has not withdrawn consent (or there is another legal ground for processing).
*   The data subject has not objected, or there are overriding legitimate grounds for the processing.
*   The data was not unlawfully processed.
*   There is no legal obligation to erase it.
*   The data was not collected in relation to information society services under Article 8(1).

Additionally, **Recital 65** states that "the further retention of the personal data should be lawful where it is necessary, for exercising the right of freedom of expression and information, for compliance with a legal obligation..." This suggests that retention (and thus refusal to erase) may be justified in these specific cases, although the Recital uses the word "should" rather than establishing a strict legal right to refuse in the same manner as Article 17 establishes the right to erasure.

**Conclusion:** The provided context does not contain an explicit list of grounds for **refusing** a deletion request. It only lists the grounds for **obligatory** erasure (Article 17(1)) and mentions that retention may be lawful for freedom of expression or legal obligations (Recital 65).

---

## Q11 - negation / role limit

**Query:** May staff working for a processor use personal data without instructions from the controller?

**Gold:** Article 29 - rank #1 -> #1

**Baseline context:** ['Article 29', 'Article 28', 'Recital 80']

**Answer 1:**

> No, staff working for a processor may not use personal data without instructions from the controller, unless required to do so by Union or Member State law.

This is based on **Article 29**, which states: "The processor and any person acting under the authority of the controller or of the processor, who has access to personal data, shall not process those data except on instructions from the controller, unless required to do so by Union or Member State law."

**Re-ranked context:** ['Article 29', 'Article 28', 'Recital 81']

**Answer 2:**

> No, staff working for a processor may not use personal data without instructions from the controller, unless required to do so by Union or Member State law.

This is based on **Article 29**, which states that "any person acting under the authority of the controller or of the processor, who has access to personal data, shall not process those data except on instructions from the controller, unless required to do so by Union or Member State law." Additionally, **Article 28(a)** requires that the processor "processes the personal data only on documented instructions from the controller... unless required to do so by Union or Member State law."

---

## Q12 - negation / scope

**Query:** In which situations does this regulation not apply at all?

**Gold:** Article 2 / Recital 18 - rank #2 -> #2

**Baseline context:** ['Recital 152', 'Recital 18', 'Recital 41']

**Answer 1:**

> Based on the provided context, the Regulation does not apply to the processing of personal data by a natural person in the course of a purely personal or household activity and thus with no connection to a professional or commercial activity.

**Source:** Recital 18

**Re-ranked context:** ['Recital 16', 'Recital 18', 'Recital 27']

**Answer 2:**

> Based on the provided context, the Regulation does not apply in the following situations:

1.  **Activities outside the scope of Union law:** Specifically, activities concerning national security, or the processing of personal data by Member States when carrying out activities in relation to the common foreign and security policy of the Union (Recital 16).
2.  **Purely personal or household activities:** The processing of personal data by a natural person in the course of a purely personal or household activity with no connection to a professional or commercial activity (Recital 18).
3.  **Data of deceased persons:** The personal data of deceased persons (Recital 27).

---

## C1 - CONTROL

**Query:** What is the maximum administrative fine for an infringement of this regulation?

**Gold:** Article 83 - rank #1 -> #1

**Baseline context:** ['Article 83', 'Recital 148', 'Article 83']

**Answer 1:**

> Based on the provided context, the maximum administrative fine for an infringement of this Regulation is **20 000 000 EUR**, or in the case of an undertaking, **4 % of the total worldwide annual turnover of the preceding financial year**, whichever is higher.

This applies to infringements of the basic principles for processing, data subjects' rights, and transfers of personal data to third countries or international organisations.

**Source:** Article 83, paragraph 5.

**Re-ranked context:** ['Article 83', 'Recital 148', 'Article 83']

**Answer 2:**

> Based on the provided context, the maximum administrative fine for an infringement of this Regulation is **20 000 000 EUR**, or in the case of an undertaking, **4 % of the total worldwide annual turnover of the preceding financial year**, whichever is higher.

This applies to infringements of the basic principles for processing, data subjects' rights, and transfers of personal data to third countries or international organisations.

**Source:** Article 83, paragraph 5.

---

## C2 - CONTROL

**Query:** What is the definition of personal data?

**Gold:** Article 4 - rank #1 -> #1

**Baseline context:** ['Article 4', 'Article 13', 'Article 4']

**Answer 1:**

> According to Article 4(1) of the provided context, ‘personal data’ means any information relating to an identified or identifiable natural person (‘data subject’). An identifiable natural person is one who can be identified, directly or indirectly, in particular by reference to an identifier such as a name, an identification number, location data, an online identifier or to one or more factors specific to the physical, physiological, genetic, mental, economic, cultural or social identity of that natural person.

**Re-ranked context:** ['Article 4', 'Article 13', 'Recital 35']

**Answer 2:**

> According to Article 4(1), “personal data” means any information relating to an identified or identifiable natural person (‘data subject’); an identifiable natural person is one who can be identified, directly or indirectly, in particular by reference to an identifier such as a name, an identification number, location data, an online identifier or to one or more factors specific to the physical, physiological, genetic, mental, economic, cultural or social identity of that natural person.

---

