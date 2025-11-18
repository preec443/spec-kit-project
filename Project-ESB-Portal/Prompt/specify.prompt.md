/speckit.specify
I want to build a web application called "Integration Effort Estimator"
for estimating Mandays (MD) based on integration features.

Input section:

1. Project name:

   - A required text field for the project name.

2. Integration features:

   - The user can add multiple rows.
   - Each row has:
     - Feature type (required, single select) with these options:
       - Use Case / API
       - Topic / Event
       - Data Synchronization
       - Batch / File
       - ESB Feature
     - Summation of Project cost (THB), numeric, non-negative.
     - Summation of Hardware cost (THB), numeric, non-negative.
   - For each row the app automatically computes:
     Estimated efforts (MD) = ROUNDDOWN(
     (Summation of Project cost + Summation of Hardware cost) / 5000
     )

3. Roles used in the output:
   - System Analyst (SA)
   - Quality Assurance (QA)
   - Back-End Developer (BE)
   - Client Support Operation (Support)

Output section:
Show a Manday table like my sketch. Columns:

- Feature
- SA (MD)
- QA (MD)
- BE (MD)
- Support (MD)
- Rest MD
- Support + Rest MD

For each feature row:
Let E = Estimated efforts (MD) for that row.
Use rounddown behavior for all multiplications (like Excel ROUNDDOWN).

1. Use Case / API

   - SA = ROUNDDOWN(E \* 0.30)
   - QA = ROUNDDOWN(E \* 0.00)
   - BE = ROUNDDOWN(E \* 0.00)
   - Support = ROUNDDOWN(E \* 0.70)
   - Rest MD = E - (SA + QA + BE + Support)
   - Support + Rest MD = Support + Rest MD

2. Topic / Event

   - Same percentages and formulas as Use Case / API.

3. Data Synchronization

   - SA = ROUNDDOWN(E \* 0.30)
   - QA = ROUNDDOWN(E \* 0.20)
   - BE = ROUNDDOWN(E \* 0.20)
   - Support = ROUNDDOWN(E \* 0.30)
   - Rest MD & Support + Rest MD same pattern.

4. Batch / File

   - SA = ROUNDDOWN(E \* 0.30)
   - QA = ROUNDDOWN(E \* 0.20)
   - BE = ROUNDDOWN(E \* 0.20)
   - Support = ROUNDDOWN(E \* 0.30)
   - Rest MD & Support + Rest MD same pattern.

5. ESB Feature
   - SA = ROUNDDOWN(E \* 0.20)
   - QA = ROUNDDOWN(E \* 0.30)
   - BE = ROUNDDOWN(E \* 0.20)
   - Support = ROUNDDOWN(E \* 0.30)
   - Rest MD & Support + Rest MD same pattern.

Also generate a summary row at the bottom:

- For each MD column, sum values from all feature rows.
- The Feature cell in the summary row should show the label "Total".

Please organize the spec using Spec Kit’s normal structure:
overview, actors, inputs, outputs, flows, edge cases, and acceptance scenarios.
